# Resumo das Trascrições

## O Projeto: Mindshare

O **Mindshare** é uma plataforma colaborativa onde equipes podem propor ideias, fornecer feedbacks e votar nas melhores sugestões. O objetivo pedagógico é demonstrar como o **GraphQL** resolve problemas comuns de APIs REST, como *overfetching* e *underfetching*, através de um único endpoint flexível.

### Stack Tecnológica

* **Runtime:** Node.js (com TypeScript).
* **Servidor API:** Apollo Server v4 integrado com Express.
* **Linguagem de Consulta:** GraphQL (via `type-graphql`).
* **ORM:** Prisma.
* **Banco de Dados:** SQLite (pela facilidade de setup local).
* **Segurança:** JWT (*JSON Web Token*) para autenticação e `bcryptjs` para hashing de senhas.

---

## Arquitetura e Conceitos Chave

### 1. Modelagem de Dados (Prisma)

Foram definidos quatro modelos principais no arquivo `schema.prisma`:

* **User:** ID, nome, e-mail, senha (opcional para convites), timestamps e relação com ideias, comentários e votos.
* **Idea:** Título, descrição, autor (relação com User), além de listas de comentários e votos.
* **Comment:** Conteúdo textual, autor e a ideia à qual pertence.
* **Vote:** Um modelo de relação única (um voto por usuário por ideia).

### 2. Resolvers e Field Resolvers

A aplicação utiliza **Field Resolvers** para lidar com relacionamentos complexos.

* **Vantagem:** Em vez de fazer múltiplas requisições REST para obter os comentários de uma ideia e os nomes de seus autores, o GraphQL resolve isso recursivamente no lado do servidor. Se o cliente pedir o autor de um comentário dentro de uma ideia, o `FieldResolver` busca essa informação apenas se solicitado.

### 3. Decorators Customizados

Foi criado o decorator `@GQLUser` para simplificar a obtenção do usuário autenticado diretamente nos argumentos das funções dos Resolvers. Ele extrai o ID do usuário do contexto do GraphQL (preenchido pelo middleware de autenticação) e busca os dados completos no banco.

---

### Fluxo de Autenticação

1. **Contexto:** Um `buildContext` intercepta o header `Authorization: Bearer <token>` em cada requisição Express.
2. **Verificação:** O token é validado. Se for válido, o ID do usuário é injetado no contexto da requisição.
3. **Middleware `isAuth`:** Um middleware do `type-graphql` protege rotas específicas (como criar ideias ou votar), impedindo o acesso de usuários não logados.

---

## Funcionalidades Implementadas (Queries e Mutations)

| Tipo         | Nome            | Descrição                                                                 |
| :----------- | :-------------- | :------------------------------------------------------------------------ |
| **Mutation** | `register`      | Cria um novo usuário e retorna tokens de acesso.                          |
| **Mutation** | `login`         | Valida credenciais e retorna JWT e Refresh Token.                         |
| **Query**    | `getUser`       | Busca detalhes de um usuário específico por ID.                           |
| **Mutation** | `createIdea`    | Permite que um usuário logado poste uma nova sugestão.                    |
| **Query**    | `listIdeas`     | Lista todas as ideias com suporte a aninhamento de comentários e votos.   |
| **Mutation** | `updateIdea`    | Atualiza título ou descrição (passando o ID por fora do objeto de dados). |
| **Mutation** | `deleteIdea`    | Remove uma ideia do banco de dados.                                       |
| **Mutation** | `createComment` | Adiciona um comentário a uma ideia específica.                            |
| **Mutation** | `toggleVote`    | Lógica de "curtir/descurtir": se o voto existe, remove; se não, cria.     |
| **Field**    | `countVotes`    | Campo calculado que retorna a quantidade total de votos de uma ideia.     |

---

## Conclusão do Projeto

Ao final, o projeto gera automaticamente um arquivo `schema.graphql` (SDL) que serve como documentação viva. O **Apollo Playground** é utilizado para testar a API, permitindo visualizar o grafo de relacionamentos e escolher exatamente quais campos retornar, otimizando a performance do front-end.
