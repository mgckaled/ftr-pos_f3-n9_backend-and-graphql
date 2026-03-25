# Arquitetura do Backend — MindShare

Documentação técnica da estrutura e responsabilidades de cada camada e arquivo do backend.

---

## Visão Geral do Fluxo de uma Requisição

```plaintext
Cliente (HTTP POST /graphql)
  │
  ├─ CORS middleware (Express)          valida a origem da requisição
  │
  ├─ express.json()                     parseia o body JSON
  │
  ├─ expressMiddleware (Apollo)         entrega a requisição ao Apollo Server
  │
  ├─ buildContext()                     extrai e valida o JWT do header Authorization
  │                                     → popula ctx.user com o ID do usuário logado
  │
  ├─ IsAuth (middleware TypeGraphQL)    bloqueia se ctx.user for undefined
  │
  ├─ Resolver                           recebe os argumentos GraphQL, chama o Service
  │
  ├─ Service                            executa a lógica de negócio, acessa o Prisma
  │
  └─ PrismaClient                       executa a query no banco SQLite
```

---

## Estrutura de Pastas

```plaintext
src/
├── index.ts                  # Bootstrap do servidor
├── graphql/
│   ├── context/              # Contexto compartilhado entre resolvers
│   └── decorators/           # Decorators customizados para parâmetros
├── middlewares/              # Middlewares de proteção de rotas
├── models/                   # ObjectTypes GraphQL (forma dos dados retornados)
├── dtos/
│   ├── input/                # InputTypes GraphQL (forma dos dados recebidos)
│   └── output/               # ObjectTypes de saída específicos (ex: login)
├── resolvers/                # Ponto de entrada das operações GraphQL
├── services/                 # Lógica de negócio e acesso ao banco
└── utils/                    # Funções utilitárias (hash, JWT)
```

---

## Arquivos

### `src/index.ts` — Bootstrap do Servidor

Ponto de entrada da aplicação. Responsável por inicializar e conectar todas as peças:

1. **CORS** é configurado para aceitar requisições apenas de `http://localhost:5173` (o frontend)
2. **`buildSchema`** do TypeGraphQL varre os resolvers e constrói o schema GraphQL em tempo de execução, emitindo também o arquivo `schema.graphql` como artefato legível
3. **`ApolloServer`** recebe o schema e é iniciado
4. **`expressMiddleware`** conecta o Apollo ao Express na rota `/graphql`, passando o `buildContext` para montar o contexto de cada requisição

---

### `src/graphql/context/index.ts` — Contexto GraphQL

Define o tipo `GraphqlContext` e a função `buildContext`, executada a cada requisição antes de qualquer resolver.

**O que faz:**

- Lê o header `Authorization: Bearer <token>`
- Verifica o JWT com `verifyJwt`
- Se válido, extrai o `id` do usuário do payload e o coloca em `context.user`
- Se inválido ou ausente, `context.user` fica `undefined`

**Por que é importante:** o contexto é o canal pelo qual a identidade do usuário trafega entre a camada HTTP e os resolvers. O `IsAuth` middleware e o `@GqlUser()` decorator dependem diretamente dele.

---

### `src/graphql/decorators/user.decorator.ts` — `@GqlUser()` Decorator

Decorator de parâmetro que resolve o usuário autenticado completo (objeto `UserModel` do banco) a partir do `context.user` (que contém apenas o ID).

**Uso nos resolvers:**

```typescript
async createIdea(@GqlUser() user: UserModel) { ... }
```

**Por que existe:** evita repetir a query `prismaClient.user.findUnique({ where: { id: context.user } })` em cada resolver que precise do objeto completo do usuário logado.

---

### `src/middlewares/auth.middleware.ts` — `IsAuth`

Middleware TypeGraphQL aplicado via `@UseMiddleware(IsAuth)` nos resolvers.

**O que faz:** verifica se `context.user` está definido. Se não estiver, lança `Error('Usuário não autenticado!')` antes que o resolver seja executado.

**Aplicação:** é decorado no nível da classe (`@Resolver`) em todos os resolvers exceto `AuthResolver`, protegendo automaticamente todas as operações do resolver.

---

### `src/models/` — ObjectTypes GraphQL

Os models definem a **forma dos dados retornados** pela API. Cada classe usa `@ObjectType()` do TypeGraphQL, e cada propriedade usa `@Field()` para ser exposta no schema GraphQL.

| Arquivo | Representa | Campos de relação |
| --------- | ----------- | ------------------- |
| `user.model.ts` | Usuário do sistema + enum `Role` | — |
| `idea.model.ts` | Ideia criada por um usuário | `author`, `comments`, `votes`, `countVotes` |
| `comment.model.ts` | Comentário em uma ideia | `author`, `idea` |
| `vote.model.ts` | Voto de um usuário em uma ideia | `user`, `idea` |

**Detalhe importante:** os campos de relação (`author`, `comments`, etc.) são declarados nos models com `{ nullable: true }` e **não são preenchidos pelo banco diretamente** — eles são resolvidos pelos `@FieldResolver` nos resolvers. O Prisma retorna apenas os campos escalares; os relacionamentos são buscados sob demanda.

---

### `src/dtos/input/` — InputTypes GraphQL

Os DTOs de entrada definem a **forma dos dados recebidos** nas mutations e queries. Usam `@InputType()` do TypeGraphQL.

| Arquivo | Classes |
| --------- | --------- |
| `auth.input.ts` | `RegisterInput` (name, email, password), `LoginInput` (email, password) |
| `user.input.ts` | `CreateUserInput` (name, email), `UpdateUserInput` (name?, role?) |
| `idea.input.ts` | `CreateIdeaInput` (title, description?), `UpdateIdeaInput` (title?, description?) |
| `comment.input.ts` | `CreateCommentInput` (content) |

**Por que separar de models:** os inputs representam o que o cliente *envia*; os models representam o que a API *retorna*. Um campo como `password` é recebido no input, mas nunca exposto no model de saída.

---

### `src/dtos/output/auth.output.ts` — OutputTypes de Autenticação

Define `RegisterOutput` e `LoginOutput`, ambos com a mesma estrutura:

- `token` — JWT de acesso (validade 1 dia)
- `refreshToken` — JWT de refresh (atualmente com a mesma validade do token)
- `user` — objeto `UserModel` do usuário autenticado

**Nota:** `refreshToken` hoje é gerado com as mesmas configurações do `token`. Uma implementação real teria validade maior e lógica separada de renovação.

---

### `src/resolvers/` — Resolvers GraphQL

Os resolvers são a **camada de entrada das operações GraphQL**. Recebem os argumentos da query/mutation, delegam para o service correspondente e retornam o resultado. Não contêm lógica de negócio.

#### `auth.resolver.ts`

- Operações públicas (sem `@UseMiddleware(IsAuth)`)
- `login` → `AuthService.login()`
- `register` → `AuthService.register()`

#### `user.resolver.ts`

- Protegido por `@UseMiddleware(IsAuth)` na classe
- `deleteUser` acessa `@Ctx()` diretamente para impedir que o usuário delete a si mesmo (`ctx.user === id`)
- Demais operações delegam diretamente ao `UserService`

#### `idea.resolver.ts`

- O resolver mais completo: possui queries, mutations e **4 field resolvers**
- `createIdea` usa `@GqlUser()` para obter o usuário completo e extrair seu `id` como `authorId`
- **Field resolvers:**
  - `author` → busca o `UserModel` pelo `authorId` da ideia
  - `comments` → busca todos os comentários da ideia
  - `votes` → busca todos os votos da ideia
  - `countVotes` → conta os votos via `VoteService.countVotes()`

#### `comment.resolver.ts`

- `createComment` recebe `ideaId` como argumento separado do DTO e usa `@GqlUser()` para o autor
- Field resolvers resolvem `idea` e `author` a partir dos IDs armazenados no comentário

#### `vote.resolver.ts`

- `toggleVote` implementa o padrão de toggle: se o voto existe, remove; se não existe, cria
- A unicidade do voto é garantida no banco pelo constraint `@@unique([userId, ideaId])` do Prisma

---

### `src/services/` — Camada de Negócio

Os services contêm a lógica de negócio e são o único ponto de acesso ao `prismaClient`. Cada service corresponde a uma entidade do domínio.

#### `auth.service.ts`

- `login`: busca o usuário por email → compara senha com bcrypt → gera tokens
- `register`: verifica duplicidade de email → faz hash da senha → cria usuário → gera tokens
- `gerenerateTokens`: método privado que centraliza a criação do `token` e `refreshToken`

#### `user.service.ts`

- `createUser`: cria usuário sem senha (fluxo de convite — o usuário não se auto-registra)
- `updateUser`: atualiza apenas os campos fornecidos (`?? undefined` evita sobrescrever com null)
- `deleteUser`: verifica existência antes de deletar, retorna `true` no sucesso

#### `idea.service.ts`

- `createIdea`: persiste a ideia com o `authorId` recebido do resolver
- `findIdeaById`: usado pelos field resolvers de `CommentResolver` e `VoteResolver` — retorna `null` se não encontrar (sem throw)
- `getIdea`: usado pelo resolver `getIdea` — lança erro se não encontrar

#### `comment.service.ts`

- `create`: valida que a ideia existe antes de criar o comentário
- `listCommentsByIdea`: retorna todos os comentários filtrando por `ideaId`

#### `vote.service.ts`

- `toggleVote`: usa o constraint composto `userId_ideaId` do Prisma para fazer o `findUnique` eficientemente
- `countVotes`: usa `prismaClient.vote.count()` — mais eficiente que buscar todos e contar no JS

---

### `src/utils/hash.ts` — Utilitários de Senha

- `hashPassword`: gera um salt com fator 10 e retorna o hash bcrypt
- `comparePassword`: compara a senha em texto plano com o hash armazenado

O fator de salt 10 é o padrão recomendado — equilibra segurança e performance.

---

### `src/utils/jwt.ts` — Utilitários de JWT

- `signJwt`: assina um payload `{ id, email }` com `JWT_SECRET` e expiration opcional
- `verifyJwt`: verifica a assinatura e retorna o payload tipado como `JwtPayload`

O `JWT_SECRET` é lido de `process.env` em tempo de execução — nunca hardcoded. A chave é carregada via `--env-file=.env` no script de desenvolvimento.

---

## Padrões Arquiteturais Utilizados

| Padrão | Onde aparece |
| -------- | ------------- |
| **Decorator** | `@ObjectType`, `@Field`, `@Resolver`, `@Mutation`, `@Query`, `@FieldResolver`, `@UseMiddleware`, `@GqlUser` |
| **DTO (Data Transfer Object)** | `src/dtos/input/` e `src/dtos/output/` |
| **Middleware** | `IsAuth` — intercepta a execução antes do resolver |
| **Service Layer** | `src/services/` — separa lógica de negócio dos resolvers |
| **Field Resolver** | Resolução lazy de relacionamentos (carregados apenas quando solicitados na query) |
| **Context** | `GraphqlContext` — transporta estado da requisição entre camadas |
| **Parameter Decorator** | `@GqlUser()` — injeção do usuário autenticado como parâmetro |
