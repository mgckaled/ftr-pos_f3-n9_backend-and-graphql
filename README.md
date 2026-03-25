<!-- markdownlint-disable -->

# Backend com GraphQL

## Objetivo

Repositório pessoal de registro, referência e suporte para fins de aprendizado, consulta e acompanhamento da disciplina de Back-end GraphQL (Nível 9), Fase 3 (Expansão de Habilidades), do curso de Pós-Graduação Tech Developer 360, desenvolvido e ministrado pela Faculdade de Tecnologia Rocketseat (FTR). 

---

## Conteúdo

### Resumo

Nesta disciplina de Back-end com GraphQL, você vai aprender na prática como construir uma API moderna e tipada utilizando Node.js, Apollo Server, TypeGraphQL e Prisma com SQLite. Ao longo das aulas, o projeto Mindshare servirá como base para explorar queries, mutations, autenticação com JWT, uso de decorators, middlewares e field resolvers, entendendo como o GraphQL otimiza consultas e simplifica a comunicação entre front-end e back-end.

- Acesso as [Trascrições de audio das Aulas](.github/docs/content/transcriptions/f3_n9_transcriptions.md)
- Acesso ao [Questionário Avaliativo](.github/docs/content/assesments/f3_n9_quiz.md) da disciplina

---

### Pesquisa 1 - GraphQL (Claude Opus 4.6)

**GraphQL** é uma linguagem de consulta para APIs criada em 2012 por Lee Byron, Dan Schafer e Nick Schrock no Facebook, tornada open source em setembro de 2015 e hoje governada pela **GraphQL Foundation** sob a Linux Foundation. Diferente do REST, o GraphQL oferece um único endpoint onde o cliente especifica exatamente os dados que precisa, eliminando problemas de overfetching e underfetching. A especificação mais recente (setembro de 2025) introduziu OneOf Input Objects, Schema Coordinates e suporte a descrições em documentos executáveis, marcando a primeira grande atualização em quatro anos. O ecossistema atual é robusto: o Apollo Client 4.0 trouxe bundles até 30% menores e arquitetura TypeScript reimaginada, o Hive Router da The Guild oferece federation open-source de alta performance, e a Apollo lançou o MCP Server para conectar agentes de IA a APIs GraphQL. Empresas como GitHub, Shopify, Netflix, Airbnb, PayPal e Twitter utilizam GraphQL em produção, e a Gartner projeta que mais de 60% das empresas terão GraphQL em produção até 2027.

No mercado de trabalho, o relatório Postman 2025 indica que GraphQL é usado por 33% dos profissionais de API, atrás de REST (93%) e Webhooks (50%). O salário médio de um desenvolvedor GraphQL nos EUA gira em torno de $110.000/ano segundo o ZipRecruiter, podendo ultrapassar $134.000 em startups de software (Wellfound). Os principais desafios em produção — como o problema N+1, caching em múltiplas camadas e segurança contra queries maliciosas — possuem soluções consolidadas: o DataLoader agrupa chamadas individuais em batch queries eficientes, caches normalizados no client podem reduzir latência em até 70%, e estratégias como query depth limiting e persisted queries previnem ataques e abuso de recursos. A tendência mais expressiva de 2025 é a convergência com inteligência artificial: o protocolo MCP posiciona o GraphQL como camada de orquestração entre agentes de IA e APIs corporativas, ampliando significativamente o escopo e a relevância da tecnologia para os próximos anos.

- [GraphQL em 2025: fundamentos, ecossistema e futuro](.github/docs/content/search/f3_n9_s1.md)

---

### Pesquisa 2 - GraphQL vs REST: anatomia prática de uma escolha arquitetural (Claude Opus 4.6)

A pesquisa comparou GraphQL e REST usando o Mindshare — uma plataforma colaborativa de ideias, comentários e votos construída com Node.js, Apollo Server v4 e Prisma — como caso de estudo concreto. A análise de três fluxos reais do sistema revelou que, no cenário mais exigente (listar ideias com comentários, autores e contagem de votos), REST exigiria potencialmente dezenas de requisições HTTP e retornaria dados desnecessários, enquanto GraphQL resolve tudo em uma única chamada ao endpoint `/graphql`, permitindo que o frontend especifique exatamente quais campos precisa. Esse contraste ilustra os dois problemas clássicos do REST — overfetching (receber dados em excesso, incluindo riscos de vazamento de campos sensíveis) e underfetching (precisar de múltiplas requisições para compor dados relacionados) — que os Field Resolvers do GraphQL eliminam ao resolver relacionamentos sob demanda, executando queries ao banco apenas quando o cliente solicita determinado campo. Contudo, a pesquisa demonstrou com igual honestidade que REST mantém superioridade em caching HTTP nativo (cada URL funciona como chave de cache para CDNs e proxies), rate limiting previsível por endpoint, upload de arquivos e APIs públicas para terceiros, onde a simplicidade e a previsibilidade do contrato REST com documentação OpenAPI/Swagger superam a flexibilidade do GraphQL.

Os casos reais de empresas em produção reforçaram que a decisão não é binária. O GitHub lançou sua API GraphQL v4 em 2016 após constatar que uma visão completa de um pull request exigia múltiplas chamadas REST, mas mantém a REST v3 até hoje por compatibilidade. A Shopify foi a mais agressiva, classificando sua REST API como "legacy" em outubro de 2024 e exigindo migração. A Netflix adotou Apollo Federation com mais de 150 subgraphs para eliminar duplicação de APIs entre iOS, Android e TV. O PayPal reduziu drasticamente o tempo de composição de telas de checkout ao eliminar round-trips REST de 700ms cada, e o Twitter/X usa GraphQL internamente mas expõe REST na API pública por simplicidade para desenvolvedores externos. No impacto ao frontend, GraphQL oferece vantagens em tipagem automática via codegen, documentação via introspection e iteração independente do backend, mas dados do State of Frontend 2024 mostram que seu uso caiu de 42,4% para 26,4% entre desenvolvedores frontend, em parte porque ferramentas como TanStack Query com REST simples resolvem muitos dos problemas que antes motivavam a adoção do GraphQL. A conclusão convergente é que ambos coexistem na indústria: REST para exposição pública, caching e simplicidade; GraphQL para composição interna, frontends com necessidades variáveis de dados e sistemas com relacionamentos complexos entre entidades — como é precisamente o caso do Mindshare.

- [GraphQL vs REST: anatomia prática de uma escolha arquitetural](.github/docs/content/search/f3_n9_s2.md)

---

### Pesquisa 3 - GraphQL em produção: segurança, performance e observabilidade além do básico (Claude Opus 4.6)

A pesquisa revela que levar uma API GraphQL como o Mindshare para produção real exige enfrentar desafios que simplesmente não existem em APIs REST tradicionais, todos derivados de uma mesma característica arquitetural: o cliente controla a forma da resposta através de um único endpoint. O problema N+1, por exemplo, é uma consequência direta do modelo de Field Resolvers — quando o Mindshare lista 50 ideias e precisa resolver o autor de cada uma, o servidor executa 101 queries ao banco sem que o desenvolvedor perceba. O DataLoader, criado por Lee Byron, resolve isso agrupando chamadas individuais em uma única query batch, mas abordagens mais recentes como o breadth-first loading da Shopify (que alcançou execução 15x mais rápida) e o Grafast de Benjie Gillam estão tornando o N+1 estruturalmente impossível. Do lado da segurança, o mesmo poder que o GraphQL dá ao cliente — escolher campos, aninhar relacionamentos, compor queries livremente — se torna uma superfície de ataque: queries com profundidade infinita podem derrubar o servidor, a introspecção expõe o schema inteiro a atacantes, e rate limiting baseado em contagem de requisições HTTP é inútil quando uma única query pode custar milhares de vezes mais que outra. A defesa requer múltiplas camadas simultâneas: depth limiting, análise de complexidade com pesos por campo (como a especificação IBM `@cost`/`@listSize` adotada pelo Apollo GraphOS), e idealmente trusted documents que restringem a execução exclusivamente a operações pré-registradas em tempo de build.

O segundo eixo crítico envolve caching, autorização granular e observabilidade — três áreas onde o GraphQL exige abordagens fundamentalmente diferentes do REST. O caching HTTP nativo, que REST ganha "de graça" com URLs únicas e GET requests, simplesmente não funciona quando tudo passa por `POST /graphql` com bodies variáveis; a compensação exige uma arquitetura de seis camadas, desde o cache normalizado do Apollo Client no frontend (que atualiza automaticamente queries relacionadas após mutations) até CDN caching com persisted queries na edge, passando por DataLoader por requisição, cache de resolver com TTL e Redis distribuído. A autorização vai muito além do middleware `isAuth` implementado no Mindshare — em produção, é preciso controlar acesso no nível de cada campo individual usando ferramentas como o decorator `@Authorized` do TypeGraphQL ou o `graphql-shield`, separando claramente autenticação (quem é você) de autorização (o que você pode fazer), idealmente reforçada por Row-Level Security no banco de dados como última linha de defesa. Por fim, a observabilidade enfrenta o fato de que o GraphQL retorna HTTP 200 mesmo para erros, tornando métricas HTTP tradicionais inúteis; o monitoramento precisa operar no nível de operation names com OpenTelemetry, rastreando latência p99 por operação, taxa de erro por campo, e distribuição de complexidade de queries — como fazem Netflix com mais de 150 subgrafos federados e Shopify com custos de query expostos diretamente nas respostas da API.

- [GraphQL em produção: segurança, performance e observabilidade além do básico](.github/docs/content/search/f3_n9_s3.md)

---

## Projeto MindShare

Plataforma colaborativa para criação, votação e discussão de ideias em equipe. Construída com uma API GraphQL no backend e uma SPA React no frontend.

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
> 
> Roteiro de testes manuais via GraphQL (curl e sandbox) em [`TESTING.md`](./TESTING.md). Referência interna testes com claude code: `claude --resume "graphql-api-manual-tests"`

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
