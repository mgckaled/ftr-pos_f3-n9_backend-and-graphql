# Testes Manuais GraphQL — MindShare

## Pré-requisitos

- Backend rodando: `npm run dev` dentro de `backend/`
- Acessar: `http://localhost:4000/graphql`
- **Usar Chrome ou Edge** — Firefox bloqueia o header `Authorization` no sandbox embutido do Apollo

## Configuração de Headers no Sandbox

Após o login, configurar os headers na aba **Headers** do sandbox:

```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImNlYzA0YThhLTcwMmQtNGJhMi04ZjU5LTc4NWFlYWMxYTcwNyIsImVtYWlsIjoiam9hb0B0ZXN0ZS5jb20iLCJpYXQiOjE3NzQzNTAwMDEsImV4cCI6MTc3NDQzNjQwMX0.QRFpt7wWw-Uhot6dvE2uE2kpfONT2jLRzm09kg8pd8A",
  "content-type": "application/json"
}
```

> O `content-type: application/json` é obrigatório — o Apollo Server v4+ bloqueia requisições sem ele como proteção contra CSRF.

---

## Bloco 1 — Autenticação (sem token)

### 1.1 Registrar usuário

```graphql
mutation {
  register(data: {
    name: "João Silva"
    email: "joao@teste.com"
    password: "123456"
  }) {
    token
    user {
      id
      name
      email
      role
    }
  }
}
```

✅ Esperado: retorna `token` e `user` com `role: member`
❌ Retest: mesmo email → deve falhar

### 1.2 Login

```graphql
mutation {
  login(data: {
    email: "joao@teste.com"
    password: "123456"
  }) {
    token
    user {
      id
      name
    }
  }
}
```

✅ Esperado: retorna `token` válido
❌ Retest: senha errada → deve retornar erro

> Copiar o `token` retornado e setar nos headers conforme seção acima.

---

## Bloco 2 — Membros/Usuários (requer token)

### 2.1 Listar membros

```graphql
query {
  listUsers {
    id
    name
    email
    role
  }
}
```

✅ Esperado: lista com pelo menos o usuário registrado

### 2.2 Buscar usuário por ID

```graphql
query {
  getUser(id: "ID_DO_USUARIO") {
    id
    name
    email
    role
    createdAt
  }
}
```

### 2.3 Criar novo membro (convite)

```graphql
mutation {
  createUser(data: {
    name: "Maria Souza"
    email: "maria@teste.com"
  }) {
    id
    name
    email
    role
  }
}
```

✅ Esperado: cria sem senha, `role` padrão `member`

### 2.4 Atualizar papel de membro

```graphql
mutation {
  updateUser(
    id: "ID_DA_MARIA"
    data: { role: admin }
  ) {
    id
    name
    role
  }
}
```

✅ Esperado: `role` atualizado para `admin`

### 2.5 Deletar membro

```graphql
mutation {
  deleteUser(id: "ID_DA_MARIA")
}
```

✅ Esperado: `true`
❌ Retest: tentar deletar a si mesmo → deve retornar erro

---

## Bloco 3 — Ideias (requer token)

### 3.1 Criar ideia completa

```graphql
mutation {
  createIdea(data: {
    title: "Melhorar o onboarding"
    description: "Criar um fluxo guiado para novos usuários"
  }) {
    id
    title
    description
    authorId
    createdAt
  }
}
```

✅ Esperado: `authorId` corresponde ao usuário logado

### 3.2 Criar ideia sem descrição (campo opcional)

```graphql
mutation {
  createIdea(data: {
    title: "Ideia rápida sem descrição"
  }) {
    id
    title
  }
}
```

### 3.3 Listar ideias com dados aninhados

```graphql
query {
  listIdeas {
    id
    title
    description
    countVotes
    author {
      name
      role
    }
    comments {
      content
      author {
        name
      }
    }
    votes {
      user {
        name
      }
    }
  }
}
```

✅ Valida todos os field resolvers de uma vez

### 3.4 Buscar ideia por ID

```graphql
query {
  getIdea(id: "ID_DA_IDEIA") {
    id
    title
    countVotes
    author {
      name
    }
  }
}
```

### 3.5 Atualizar ideia

```graphql
mutation {
  updateIdea(
    id: "ID_DA_IDEIA"
    data: { title: "Melhorar o onboarding v2" }
  ) {
    id
    title
    updatedAt
  }
}
```

---

## Bloco 4 — Votos (requer token)

### 4.1 Votar (toggle ON)

```graphql
mutation {
  toggleVote(ideaId: "ID_DA_IDEIA")
}
```

✅ Esperado: `true` (voto criado)

### 4.2 Verificar contagem

```graphql
query {
  getIdea(id: "ID_DA_IDEIA") {
    title
    countVotes
  }
}
```

✅ Esperado: `countVotes: 1`

### 4.3 Remover voto (toggle OFF)

```graphql
mutation {
  toggleVote(ideaId: "ID_DA_IDEIA")
}
```

✅ Esperado: `true` (voto removido); `countVotes` volta a `0`

---

## Bloco 5 — Comentários (requer token)

### 5.1 Criar comentário

```graphql
mutation {
  createComment(
    ideaId: "ID_DA_IDEIA"
    data: { content: "Ótima ideia! Podemos usar isso no Q3." }
  ) {
    id
    content
    author {
      name
    }
    idea {
      title
    }
  }
}
```

✅ Valida os field resolvers do comentário

### 5.2 Ver comentários na listagem de ideias

```graphql
query {
  listIdeas {
    title
    countVotes
    comments {
      content
      author { name }
      createdAt
    }
  }
}
```

---

## Bloco 6 — Proteção de rotas

### 6.1 Sem token (remover o header Authorization)

```graphql
query {
  listIdeas {
    id
  }
}
```

✅ Esperado: erro `"Usuário não autenticado!"`

### 6.2 Token inválido

Setar `"Authorization": "Bearer tokeninvalido"` e repetir qualquer query protegida.

✅ Esperado: erro de JWT inválido

---

## Bloco 7 — Deletar ideia

### 7.1 Deletar

```graphql
mutation {
  deleteIdea(id: "ID_DA_IDEIA")
}
```

✅ Esperado: `true`

### 7.2 Confirmar deleção

```graphql
query {
  getIdea(id: "ID_DA_IDEIA") {
    id
  }
}
```

✅ Esperado: erro ou `null`

---

## Ordem Recomendada de Execução

```text
register → login → (setar token + content-type nos headers)
→ createIdea × 2 → listIdeas (ver dados aninhados)
→ toggleVote → getIdea (verificar countVotes) → toggleVote (desfaz)
→ createComment → listIdeas (ver comentários)
→ createUser → updateUser → deleteUser
→ updateIdea → deleteIdea
→ testar sem token / token inválido
```

---

## Referência Rápida das Operações

| Categoria      | Operações                                                              | Auth |
|----------------|------------------------------------------------------------------------|------|
| Autenticação   | `register`, `login`                                                    | Não  |
| Usuários       | `listUsers`, `getUser`, `createUser`, `updateUser`, `deleteUser`       | Sim  |
| Ideias         | `listIdeas`, `getIdea`, `createIdea`, `updateIdea`, `deleteIdea`       | Sim  |
| Votos          | `toggleVote`                                                           | Sim  |
| Comentários    | `createComment`                                                        | Sim  |

---

## Alternativa ao Sandbox Embutido

Caso o sandbox do Apollo apresente problemas (ex: Firefox bloqueando headers), use o **Insomnia**:

1. Criar requisição `POST` para `http://localhost:4000/graphql`
2. Body: tipo **GraphQL**
3. Headers:
   - `Authorization: Bearer SEU_TOKEN`
   - `content-type: application/json`
