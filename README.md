<!-- markdownlint-disable -->

# Backend com GraphQL

claude --resume "graphql-api-manual-tests"

Nesta disciplina de Back-end com GraphQL, você vai aprender na prática como construir uma API moderna e tipada utilizando Node.js, Apollo Server, TypeGraphQL e Prisma com SQLite. Ao longo das aulas, o projeto Mindshare servirá como base para explorar queries, mutations, autenticação com JWT, uso de decorators, middlewares e field resolvers, entendendo como o GraphQL otimiza consultas e simplifica a comunicação entre front-end e back-end.

## Conteúdo

- Acesso as [Trascrições de audio das Aulas](.github/docs/content/transcriptions/f3_n9_transcriptions.md)
- Acesso ao [Questionário Avaliativo](.github/docs/content/assesments/f3_n9_quiz.md) da disciplina

### Pesquisa GraphQL (Claude Opus 4.6)

**GraphQL** é uma linguagem de consulta para APIs criada em 2012 por Lee Byron, Dan Schafer e Nick Schrock no Facebook, tornada open source em setembro de 2015 e hoje governada pela **GraphQL Foundation** sob a Linux Foundation. Diferente do REST, o GraphQL oferece um único endpoint onde o cliente especifica exatamente os dados que precisa, eliminando problemas de overfetching e underfetching. A especificação mais recente (setembro de 2025) introduziu OneOf Input Objects, Schema Coordinates e suporte a descrições em documentos executáveis, marcando a primeira grande atualização em quatro anos. O ecossistema atual é robusto: o Apollo Client 4.0 trouxe bundles até 30% menores e arquitetura TypeScript reimaginada, o Hive Router da The Guild oferece federation open-source de alta performance, e a Apollo lançou o MCP Server para conectar agentes de IA a APIs GraphQL. Empresas como GitHub, Shopify, Netflix, Airbnb, PayPal e Twitter utilizam GraphQL em produção, e a Gartner projeta que mais de 60% das empresas terão GraphQL em produção até 2027.

No mercado de trabalho, o relatório Postman 2025 indica que GraphQL é usado por 33% dos profissionais de API, atrás de REST (93%) e Webhooks (50%). O salário médio de um desenvolvedor GraphQL nos EUA gira em torno de $110.000/ano segundo o ZipRecruiter, podendo ultrapassar $134.000 em startups de software (Wellfound). Os principais desafios em produção — como o problema N+1, caching em múltiplas camadas e segurança contra queries maliciosas — possuem soluções consolidadas: o DataLoader agrupa chamadas individuais em batch queries eficientes, caches normalizados no client podem reduzir latência em até 70%, e estratégias como query depth limiting e persisted queries previnem ataques e abuso de recursos. A tendência mais expressiva de 2025 é a convergência com inteligência artificial: o protocolo MCP posiciona o GraphQL como camada de orquestração entre agentes de IA e APIs corporativas, ampliando significativamente o escopo e a relevância da tecnologia para os próximos anos.

- [GraphQL em 2025: fundamentos, ecossistema e futuro](.github/docs/content/search/f3_n9_search.md)

## Projeto MindShare

Plataforma colaborativa para criação, votação e discussão de ideias em equipe. Construída com uma API GraphQL no backend e uma SPA React no frontend.

---

### Requisitos

- Node.js >= 22
- pnpm (frontend) / npm (backend)

### Visão Geral

| Camada    | Tecnologia principal                         | Porta |
|-----------|----------------------------------------------|-------|
| Backend   | Node.js · Apollo Server · TypeGraphQL · Prisma | 4000  |
| Frontend  | React 19 · Vite · Apollo Client · Zustand    | 5173  |

---

### Funcionalidades

- Autenticação com JWT (registro e login)
- CRUD de ideias com título e descrição
- Sistema de votos por ideia (toggle)
- Comentários por ideia
- Gerenciamento de membros com controle de papéis (owner, admin, member, viewer)
- Rotas protegidas e redirecionamento automático conforme autenticação

---

### Estrutura do Repositório

```plaintext
.
├── backend/      # API GraphQL (Apollo Server + Prisma)
└── frontend/     # SPA React (Vite + Apollo Client)
```

---

### Backend

#### Stack

- **Runtime**: Node.js 22 + TypeScript (`tsx`)
- **GraphQL**: Apollo Server v5 + TypeGraphQL v2
- **HTTP**: Express v5
- **ORM**: Prisma v6 (SQLite)
- **Auth**: JWT + bcryptjs

> Documentação técnica detalhada de cada camada, padrões arquiteturais e fluxo de requisição em [`backend/ARCHITECTURE.md`](./backend/ARCHITECTURE.md).
> Roteiro de testes manuais via GraphQL (curl e sandbox) em [`TESTING.md`](./TESTING.md).

#### Modelos de Dados

| Modelo    | Campos principais                                              |
|-----------|----------------------------------------------------------------|
| `User`    | id, name, email, password, role (owner/admin/member/viewer)   |
| `Idea`    | id, title, description, authorId                              |
| `Vote`    | id, userId, ideaId *(unique por par)*                         |
| `Comment` | id, content, authorId, ideaId                                 |

#### Resolvers GraphQL

| Resolver          | Operações                                              |
|-------------------|--------------------------------------------------------|
| `AuthResolver`    | `login`, `register`                                    |
| `IdeaResolver`    | `createIdea`, `updateIdea`, `deleteIdea`, `listIdeas`, `getIdea` |
| `UserResolver`    | `createUser`, `updateUser`, `deleteUser`, `listUsers`, `getUser` |
| `CommentResolver` | `createComment`                                        |
| `VoteResolver`    | `toggleVote`                                           |

Todos os resolvers (exceto auth) são protegidos pelo middleware `IsAuth`.

#### Configuração

```bash
cd backend
cp .env.example .env   # configure DATABASE_URL e JWT_SECRET
npm install
npm run generate       # gera o Prisma Client
npm run migrate        # aplica as migrations
npm run dev            # inicia em http://localhost:4000/graphql
```

#### Scripts

| Script            | Descrição                          |
|-------------------|------------------------------------|
| `npm run dev`     | Inicia o servidor com hot-reload   |
| `npm run migrate` | Executa migrations do Prisma       |
| `npm run generate`| Gera o Prisma Client               |
| `npm run seed`    | Popula o banco com dados iniciais  |

---

### Frontend

#### Stack

- **Framework**: React 19 + TypeScript
- **Build**: Vite 7
- **Roteamento**: React Router DOM v7
- **GraphQL**: Apollo Client v4
- **Estado global**: Zustand (com persistência em localStorage)
- **UI**: Radix UI + Tailwind CSS v3 + shadcn/ui
- **Notificações**: Sonner
- **Ícones**: Lucide React

#### Páginas

| Rota        | Componente                    | Acesso       |
|-------------|-------------------------------|--------------|
| `/login`    | `pages/Auth/Login.tsx`        | Público      |
| `/signup`   | `pages/Auth/Signup.tsx`       | Público      |
| `/`         | `pages/Ideias/index.tsx`      | Autenticado  |
| `/members`  | `pages/Members/index.tsx`     | Autenticado  |

#### Fluxo de Autenticação

1. Login/registro envia mutation GraphQL e recebe um JWT
2. O token é armazenado no Zustand (persiste em `localStorage`)
3. O Apollo Client injeta o token automaticamente via `setContext` em cada requisição

#### Configuração

```bash
cd frontend
pnpm install
pnpm dev    # inicia em http://localhost:5173
```

> O frontend espera o backend rodando em `http://localhost:4000/graphql`.

---

### Variáveis de Ambiente

#### Backend (`.env`)

```env
DATABASE_URL="file:./dev.db"
JWT_SECRET="sua_chave_secreta"
```

---
