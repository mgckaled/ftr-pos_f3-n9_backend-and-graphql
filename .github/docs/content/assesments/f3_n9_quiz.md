<!-- markdownlint-disable -->

# Questionário Avaliativo

## Questão 1: O que é um resolver no contexto do GraphQL?

### ✅ Resposta Correta: Alternativa 4
**Uma função que resolve e retorna os dados de uma query ou mutation**

### Explicação Detalhada
Um resolver é fundamentalmente uma função que implementa a lógica necessária para buscar ou processar dados em resposta a uma query ou mutation específica do GraphQL. No projeto Mindshare, essa definição fica evidente em todos os momentos em que resolvers são criados. Quando o instrutor implementa o primeiro resolver simples com um "Hello World", demonstra que um resolver é uma função que executa quando uma query é chamada e retorna o resultado esperado. Posteriormente, nos resolvers mais complexos como `register`, `login`, e `createIdea`, cada um contém toda a lógica de negócio necessária (validações, operações no banco de dados, geração de tokens) e retorna os dados estruturados de acordo com o tipo GraphQL definido.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Um tipo de banco de dados"): Resolvers não são bancos de dados, mas sim código que *usa* bancos de dados para buscar informações. Essa confusão fundamental de categorias torna essa resposta completamente inadequada.

**Alternativa 2** ("Um tipo de autenticação"): Embora resolvers possam *usar* autenticação através de middlewares (como vemos com o `isAuth` na Aula 4), eles não são em si um sistema de autenticação. A autenticação é um recurso implementado através de middlewares que protegem resolvers específicos.

**Alternativa 3** ("Um arquivo de configuração"): Resolvers são código executável sob a forma de funções TypeScript, não arquivos de configuração como `tsconfig.json` ou `schema.prisma`. Essa resposta confunde a implementação com a configuração da aplicação.

---

## Questão 2: Qual biblioteca foi usada durante as aulas para criar o servidor GraphQL do projeto?

### ✅ Resposta Correta: Alternativa 1
**Apollo Server**

### Explicação Detalhada
Apollo Server é a biblioteca específica utilizada para criar o servidor GraphQL no projeto Mindshare. Isso fica evidente desde a primeira aula, quando o instrutor menciona explicitamente: "vou dar um install no Apollo Server e no GraphQL" como os primeiros passos para montar o servidor. Apollo Server é responsável por orquestrar todo o funcionamento do GraphQL, gerenciando queries, mutations, resolvers, e fornecendo o Apollo Playground — a interface visual que o instrutor usa para testar todas as operações. Na stack tecnológica documentada, Apollo Server v4 é listado como o "Servidor API" principal. Inicialmente, o servidor é criado usando o `StartStandaloneServer` do Apollo, e posteriormente é integrado com Express na Aula 2 para obter maior flexibilidade no gerenciamento de middlewares e rotas.

### Por que as outras alternativas estão incorretas

**Alternativa 2** (Fastify): Fastify é um framework web alternativo que não foi mencionado em nenhum momento durante as aulas. Enquanto Fastify poderia potencialmente ser usado como base para um servidor GraphQL, ele não foi escolhido para este projeto e não aparece em nenhuma parte do código ou documentação.

**Alternativa 3** (Express): Express é um framework web importante no projeto, mas não é a biblioteca que *cria* o servidor GraphQL em si. Express funciona como middleware integrado *com* o Apollo Server através da biblioteca `@as-integrations/express`, conforme explicado na Aula 2. Express fornece capacidades HTTP tradicionais, enquanto Apollo Server fornece a implementação GraphQL.

**Alternativa 4** (Koa): Koa é outro framework web que poderia ser utilizado, mas não foi mencionado nas aulas. Não há qualquer referência a Koa na documentação ou nas transcrições do projeto Mindshare.

---

## Questão 3: Qual é a função principal de uma mutation?

### ✅ Resposta Correta: Alternativa 1
**Criar ou alterar dados**

### Explicação Detalhada
Uma mutation no GraphQL é uma operação que tem como propósito modificar dados — seja criando novos registros, atualizando dados existentes ou deletando informações. Essa é uma distinção fundamental entre queries (que leem dados) e mutations (que modificam dados) em GraphQL. No projeto Mindshare, isso fica absolutamente claro ao observar todas as mutations implementadas: `register` (cria um novo usuário), `login` (autentica e retorna dados modificados), `createIdea` (cria uma nova ideia), `updateIdea` (altera uma ideia existente), `deleteIdea` (remove uma ideia), `createComment` (cria um comentário), e `toggleVote` (adiciona ou remove um voto). Todas essas operações compartilham a característica comum de alterar o estado dos dados no banco de dados. Quando o instrutor explica que está criando uma mutation para cadastro de usuário, deixa claro que essa é a forma correta de realizar operações que modificam o banco de dados.

### Por que as outras alternativas estão incorretas

**Alternativa 2** ("Gerar schemas"): Schemas são definições de tipos e estrutura de dados que descrevem a forma dos dados em GraphQL. Esses schemas são gerados uma única vez através do `buildSchema` do type-graphql na inicialização da aplicação. O arquivo `schema.graphql` é gerado automaticamente e documentado pela aplicação como um todo, não por cada mutation executada.

**Alternativa 3** ("Apagar cache"): Embora cache seja um conceito importante em APIs, as mutations não têm como função primária gerenciar ou limpar cache. As mutations focam unicamente em modificar dados, e qualquer invalidação de cache seria um efeito colateral secundário tratado pela aplicação cliente, não pela mutation em si.

**Alternativa 4** ("Consultar dados"): Esta é justamente a função de *queries*, não de mutations. Essa é a distinção chave entre os dois conceitos em GraphQL. Queries são operações de leitura que não modificam estado, enquanto mutations são operações de escrita que alternam dados.

---

## Questão 4: Qual é a função do Field Resolver?

### ✅ Resposta Correta: Alternativa 2
**Resolver campos de relacionamento, como autor de uma ideia**

### Explicação Detalhada
Um Field Resolver é um mecanismo especializado do GraphQL que é acionado automaticamente quando o cliente solicita um campo de relacionamento que não está diretamente disponível nos dados iniciais retornados. A sua função é popular esse campo buscando os dados relacionados do banco de dados ou de outras fontes. No projeto Mindshare, o Field Resolver fica absolutamente evidente quando o instrutor precisa resolver o campo `author` (autor) de uma ideia. Inicialmente, apenas o `authorId` é armazenado. Porém, quando o cliente solicita dados completos do autor (como `name` e `email`), o Field Resolver entra em ação. O instrutor explica: "O GraphQL vai tentar devolver as informações, e quando ele não encontra algum dado, ele vai tentar executar uma função do nosso FieldResolver para poder popular o retorno desse dado". Esse padrão se repete para resolver comentários aninhados, votos e seus autores, demonstrando como Field Resolvers eliminam problemas de overfetching e underfetching típicos de APIs REST.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Subir o servidor"): É responsabilidade do Apollo Server e do Express, não do Field Resolver. A inicialização do servidor é feita através de `app.listen()` e `server.start()`, conforme visto nas aulas de setup inicial. Field Resolvers não têm nenhuma responsabilidade com infraestrutura ou inicialização.

**Alternativa 3** ("Atualizar tokens expirados"): É função do mecanismo de refresh token, não do Field Resolver. O projeto implementa um `refreshToken` com expiração de um dia para permitir renovação de tokens de acesso quando expiram. Isso é totalmente separado de relacionamentos de dados.

**Alternativa 4** ("Validar inputs do usuário"): É responsabilidade de validadores de entrada, implementados usando `class-validator` (mencionado mas desativado com `validate: false`). Validação ocorre nos DTOs como `RegisterInput` e `CreateIdeaInput`. Field Resolvers trabalham com dados já recebidos e validados.

---

## Questão 5: Qual é o propósito do JWT no projeto?

### ✅ Resposta Correta: Alternativa 4
**Fazer autenticação e geração de tokens**

### Explicação Detalhada
JWT (JSON Web Token) é implementado no projeto Mindshare especificamente para criar um sistema robusto de autenticação e autorização. O JWT funciona como um mecanismo seguro de credencial que permite ao servidor verificar a identidade do usuário sem necessidade de armazenar sessões no servidor. Na Aula 4, o instrutor dedica grande parte da explicação ao fluxo de autenticação usando JWT. Ele cria a função `signJWT` que recebe um payload contendo o ID e email do usuário, junto com um secret armazenado em variáveis de ambiente (`.env`). Essa função gera um token criptografado que é retornado ao cliente no momento do login e do registro. O instrutor explica que o token tem uma expiração curta (15 minutos) para segurança, e também cria um refresh token com expiração mais longa (um dia) para permitir renovação sem forçar novo login. Quando o cliente faz uma requisição, envia o token no header `Authorization: Bearer <token>`. O middleware `buildContext` intercepta essa requisição, extrai o token, valida-o usando `verifyJWT`, e se for válido, injeta o ID do usuário no contexto GraphQL. Isso permite que resolvers saibam qual usuário está fazendo a requisição sem armazenar dados de sessão no servidor.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Criar schemas GraphQL"): Schemas GraphQL são criados através do type-graphql e da função `buildSchema` que compila decoradores TypeScript em definições GraphQL. O JWT não participa desse processo de definição de tipos e não gera o arquivo `schema.graphql`.

**Alternativa 2** ("Validar tipos de dados"): Validação de tipos em GraphQL é feita através de decoradores como `@Field()`, `@InputType()` e opcionalmente `class-validator`. O JWT não valida estrutura de dados; valida apenas a identidade e autenticação do usuário.

**Alternativa 3** ("Criptografar o banco de dados"): JWT usa criptografia internamente para assinar tokens, mas não é responsável por criptografar dados no banco de dados. Criptografia de dados sensíveis como senhas é feita por bibliotecas específicas como `bcryptjs`, conforme implementado na função `hashPassword`.

---

## Questão 6: Qual é a principal diferença entre REST e GraphQL?

### ✅ Resposta Correta: Alternativa 3
**GraphQL utiliza um único endpoint e o cliente define os dados desejados**

### Explicação Detalhada
Essa é a distinção fundamental entre os dois paradigmas de design de API. Enquanto REST requer múltiplos endpoints cada um retornando um conjunto fixo de dados, GraphQL concentra toda a funcionalidade em um único endpoint e permite que o cliente especifique precisamente quais campos deseja receber em cada requisição. Na Aula 1, o instrutor explica isso como motivação para criar o projeto: "Diferente do REST, onde a gente precisa de vários endpoints que retornam dados fixos, no GraphQL a gente só tem um endpoint e o cliente especifica exatamente os dados que quer retornar". Para ilustrar a diferença, ele oferece um exemplo concreto: com REST seria necessário "bater em vários endpoints para poder juntar as informações. Por exemplo, no barra Users, no barra Ideias, no barra Comments". Isso demonstra dois problemas que GraphQL resolve. Primeiro está o overfetching — quando REST retorna mais dados do que necessário (todos os campos de um usuário quando você só quer o nome). Segundo está o underfetching — quando você precisa fazer várias requisições separadas para reunir informações relacionadas. Com GraphQL, uma única requisição pode especificar exatamente quais campos de quais entidades relacionadas você quer.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("REST retorna apenas JSON"): REST pode retornar JSON, XML, ou outros formatos. GraphQL também retorna JSON como formato padrão. Ambas usam JSON como formato padrão, então isso não é uma distinção meaningful entre elas.

**Alternativa 2** ("REST utiliza apenas um endpoint"): É o oposto da verdade. REST é caracterizado justamente pelo uso de múltiplos endpoints, cada um representando um recurso diferente (users, ideas, comments, votes, etc.). Dizer que REST usa apenas um endpoint inverte o conceito fundamental.

**Alternativa 4** ("GraphQL precisa de vários endpoints"): É incorreta. Um dos pontos fortes centrais do GraphQL é que toda a API funciona através de um único endpoint (geralmente `/graphql`). Implementar GraphQL com múltiplos endpoints tiraria toda vantagem da tecnologia.

---

## Questão 7: O que é o GraphQL?

### ✅ Resposta Correta: Alternativa 1
**Uma linguagem de consulta criada pelo Facebook**

### Explicação Detalhada
GraphQL é fundamentalmente uma linguagem de consulta e especificação aberta que foi desenvolvida pelo Facebook e tornada pública em 2015. Na Aula 1, o instrutor deixa isso cristalino em sua definição inicial: "GraphQL é uma linguagem de consulta criada pelo Facebook em 2012 e aberta em 2015". Uma linguagem de consulta é um sistema que permite ao cliente articular exatamente quais dados ele deseja, diferente de um framework que fornece componentes prontos. GraphQL é uma especificação que define como as consultas devem ser estruturadas e como os servidores devem responder a elas. É agnóstico a linguagem de programação e banco de dados — você pode usar GraphQL com Node.js, Python, Java, com SQL, NoSQL, ou qualquer combinação. No contexto do projeto Mindshare, GraphQL serve como a camada de comunicação entre o servidor (implementado com Apollo Server) e o cliente. O cliente envia queries estruturadas em GraphQL especificando exatamente quais campos deseja, e o servidor interpreta essa linguagem através dos resolvers para retornar precisamente esses dados.

### Por que as outras alternativas estão incorretas

**Alternativa 2** ("Um framework para front-end"): Embora GraphQL seja frequentemente usado em aplicações front-end para buscar dados, não é um framework de interface gráfica. Ferramentas como React, Vue e Angular são frameworks de front-end. GraphQL é agnóstico a plataforma — funciona em aplicações web, mobile, desktop ou até em comunicações entre servidores.

**Alternativa 3** ("Um banco de dados relacional"): Um banco de dados relacional como PostgreSQL, MySQL ou SQLite é um sistema de armazenamento. No projeto Mindshare, SQLite é o banco relacional, mas GraphQL é apenas a linguagem que permite consultar dados — ele não armazena dados nem funciona como banco. GraphQL trabalha "em frente" ao banco de dados.

**Alternativa 4** ("Uma biblioteca de autenticação"): Bibliotecas de autenticação como JWT ou OAuth são sistemas específicos para validar identidade de usuários. Enquanto GraphQL pode ser usado em uma arquitetura que inclui autenticação, GraphQL em si não fornece nenhum mecanismo de autenticação. A autenticação é implementada separadamente através de JWT e bcryptjs.

---

## Questão 8: O que o middleware de autenticação faz?

### ✅ Resposta Correta: Alternativa 4
**Valida se o usuário está autenticado antes de executar resolvers**

### Explicação Detalhada
Um middleware de autenticação é um componente intermediário que intercepta requisições antes que elas cheguem aos resolvers, verificando se o usuário que faz a requisição está devidamente autenticado. Se o usuário não estiver autenticado, o middleware bloqueia a execução do resolver; se estiver, permite que a lógica do resolver prossiga normalmente. Na Aula 4, o instrutor implementa esse conceito através da função `isAuth`. Ele explica: "Um middleware do `type-graphql` protege rotas específicas (como criar ideias ou votar), impedindo o acesso de usuários não logados". O fluxo funciona em camadas. Quando uma requisição chega ao servidor, a função `buildContext` intercepta o header `Authorization: Bearer <token>`, extrai o token JWT, valida-o usando `verifyJWT`, e se for válido, injeta o ID do usuário no contexto GraphQL. Depois, o middleware `isAuth` entra em ação, verificando se existe um `context.user`. Se não houver, executa um `throw newError` com mensagem "Usuário não autenticado", bloqueando a execução. Se houver, chama `next()` permitindo que o resolver continue. Isso é particularmente importante para operações sensíveis como criar ideias, comentar, ou votar.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Cria novos usuários"): A criação de usuários é responsabilidade da mutation `register`, que recebe dados de entrada, valida se o email já existe, faz hash da senha, e persiste os dados no banco. A mutation `register` não precisa nem de autenticação porque um novo usuário não tem credenciais ainda.

**Alternativa 2** ("Faz log de erros"): Logging é uma preocupação separada. Enquanto um middleware *pode* fazer logging como efeito colateral, sua função principal não é essa. Logging seria implementado em um middleware diferente ou em um serviço de observabilidade dedicado.

**Alternativa 3** ("Conecta ao banco de dados"): É responsabilidade do Prisma Client. O middleware de autenticação não faz conexões com banco de dados — valida tokens e verifica se há um usuário no contexto. Se um resolver precisar de dados do banco, usa Prisma Client diretamente.

---

## Questão 9: O que o decorator @GQLUser faz no projeto?

### ✅ Resposta Correta: Alternativa 3
**Recupera o usuário autenticado a partir do contexto**

### Explicação Detalhada
O decorator `@GQLUser` é uma solução elegante criada na Aula 5 especificamente para simplificar como os resolvers acessam informações do usuário autenticado. Este é um exemplo de um padrão customizado que torna o código mais limpo e expresivo. O decorator resolve um problema prático: quando você cria uma mutation como `createIdea`, você precisa saber qual usuário está fazendo essa requisição para vincular a ideia ao seu criador. Sem o decorator, você teria que extrair manualmente o ID do contexto, depois buscar o usuário no banco de dados dentro de cada resolver. O decorator `@GQLUser` automatiza todo esse processo. Tecnicamente, é implementado usando `createParameterDecorator` do type-graphql. Quando você coloca `@GQLUser() user: User` como parâmetro de um resolver, o decorator intercepta a execução e executa uma função que: verifica se existe um `context.user` (injetado pelo middleware de autenticação); se não existir, retorna `null`; se existir, busca o usuário completo no banco usando Prisma com o ID do contexto; retorna o objeto User completo com todos os seus dados. Na Aula 5, quando o instrutor cria a mutation `createIdea`, ele demonstra isso: depois de definir `@GQLUser() user: User`, pode simplesmente usar `user.id` para vincular a ideia ao autor.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Define um novo tipo GraphQL"): O `@GQLUser` não define tipos — ele consome um tipo já existente (User). Os tipos GraphQL são definidos através de decoradores como `@ObjectType()` no UserModel e `@InputType()` nos DTOs. O `@GQLUser` é um parameter decorator que trabalha com tipos já existentes, não cria novos.

**Alternativa 2** ("Cria um novo usuário no banco"): O `@GQLUser` apenas *recupera* dados de um usuário que já existe e está autenticado. Criar novos usuários é responsabilidade da mutation `register`. O decorator nunca modifica dados; ele apenas os acessa.

**Alternativa 4** ("Atualiza o token JWT"): O decorator trabalha com autenticação que já foi feita — assume que o token JWT já foi validado e o usuário já está no contexto. A atualização de tokens seria tratada em uma mutation separada, não aqui. O `@GQLUser` é consumidor de dados de autenticação já estabelecidos.

---

## Questão 10: O que é o decorator @UseMiddleware?

### ✅ Resposta Correta: Alternativa 4
**Define uma função como middleware no resolver**

### Explicação Detalhada
O decorator `@UseMiddleware` do type-graphql permite que você aplique funções middleware a resolvers específicos ou a uma classe inteira de resolvers. Um middleware no contexto de GraphQL é uma função que executa antes do resolver principal, permitindo validar, processar, ou bloquear a requisição conforme necessário. Na Aula 4, o instrutor implementa isso de forma clara quando cria o middleware `isAuth` e precisa proteger suas rotas. Ele explica que existem duas estratégias: colocar o decorator no topo de uma classe resolver (protege todos os métodos), ou colocar logo depois de um `@Query()` ou `@Mutation()` (protege apenas aquele método). O fluxo de uma requisição é: quando um cliente faz uma requisição a um resolver protegido por `@UseMiddleware(isAuth)`, a função `isAuth` executa primeiro. Dentro dessa função, verifica: "Existe um usuário autenticado no contexto?" Se não, lança um erro bloqueando a execução. Se existir, chama `next()` permitindo que o resolver prossiga. No projeto Mindshare, todos os resolvers que fazem modificações de dados usam `@UseMiddleware(isAuth)` para garantir que apenas usuários autenticados possam executar essas operações.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Cria um novo schema"): Schemas GraphQL são definições de tipos criadas por decoradores como `@ObjectType()`, `@InputType()`, e pela função `buildSchema()`. O `@UseMiddleware` não cria ou modifica schemas — apenas protege rotas já definidas no schema.

**Alternativa 2** ("Serve para criar novas tabelas no banco"): Criação de tabelas é responsabilidade do Prisma e suas migrations. O middleware `@UseMiddleware` trabalha no nível da API GraphQL, não no nível de banco de dados. Ele não toca em estrutura de tabelas — apenas valida se uma requisição é autorizada antes de executar a lógica do resolver.

**Alternativa 3** ("Substitui a função resolver"): Middleware não substitui resolvers, ele os envolve. Funciona como um interceptador — executa antes do resolver, valida algo, e depois permite ou bloqueia a execução. O resolver nunca é substituído; é precedido pelo middleware.

---

## Questão 11: No projeto desenvolvido em aula, qual biblioteca foi utilizada para aplicar tipagem ao GraphQL?

### ✅ Resposta Correta: Alternativa 3
**TypeGraphQL**

### Explicação Detalhada
TypeGraphQL é a biblioteca fundamental que permite aos desenvolvedores definir tipos GraphQL usando classes TypeScript com decoradores, criando uma experiência totalmente type-safe e integrada com a linguagem TypeScript. Esta é uma escolha arquitetural importante que o instrutor faz desde a Aula 2. Quando explica as bibliotecas necessárias, ele destaca: "E o Type GraphQL, que é uma das bibliotecas mais importantes para a gente poder utilizar as chipagens do GraphQL dentro do nosso Node também". O propósito de TypeGraphQL é eliminar a duplicação de código e garantir que as definições de tipos sejam sincronizadas entre TypeScript e GraphQL. Sem TypeGraphQL, você teria que: definir interfaces TypeScript para seus tipos; definir strings SDL (Schema Definition Language) separadas em GraphQL; manter ambas em sincronização manualmente. Com TypeGraphQL, você faz tudo em um único lugar. Quando o instrutor cria o `UserModel`, usa decoradores TypeGraphQL como `@ObjectType()`, `@Field()`, e `@ID` em uma classe TypeScript. Essa classe é simultaneamente uma classe TypeScript válida (com tipos que o compilador TypeScript entende) e a definição completa do tipo GraphQL (que o Apollo Server usa para gerar o schema). Na Aula 5, quando cria `RegisterInput` como um `@InputType()`, está dizendo ao GraphQL exatamente quais campos esperar, com que tipos e se são obrigatórios. Quando cria `IdeaModel` com `@ObjectType()` e múltiplos `@Field()` com seus tipos, TypeGraphQL traduz automaticamente isso para tipos GraphQL válidos. Finalmente, quando chama `buildSchema` com o array de resolvers, TypeGraphQL analisa todos os decoradores e gera automaticamente o arquivo `schema.graphql`.

### Por que as outras alternativas estão incorretas

**Alternativa 1** ("Apollo Types"): Não é uma biblioteca real ou relevante para tipagem em GraphQL. Apollo fornece várias ferramentas (Apollo Server, Apollo Client, Apollo Code Generator), mas "Apollo Types" não é uma delas. Essa é uma distração criada para confundir com nomes similares.

**Alternativa 2** ("TypeScript"): É a linguagem na qual TypeGraphQL funciona, mas não é a biblioteca responsável por aplicar tipagem ao GraphQL. TypeScript adiciona tipos estáticos à linguagem JavaScript, mas por si só, TypeScript não sabe nada sobre GraphQL. A integração entre TypeScript e GraphQL é precisamente o que TypeGraphQL fornece. TypeScript é o idioma; TypeGraphQL é a ponte que conecta esse idioma a GraphQL.

**Alternativa 4** ("GraphType"): Não é uma biblioteca real. Parece ser um nome inventado que combina "Graph" e "Type" para parecer plausível, mas não existe uma biblioteca com esse nome no ecossistema GraphQL ou Node.js. No arquivo package.json fornecido, não há qualquer menção a "GraphType", e nas aulas, o instrutor nunca refere-se a qualquer biblioteca com esse nome.

---

## Resumo de Desempenho

| Questão | Tema                   | Resposta      | Dificuldade |
| ------- | ---------------------- | ------------- | ----------- |
| 1       | Conceitos Fundamentais | Alternativa 4 | Média       |
| 2       | Stack Tecnológico      | Alternativa 1 | Fácil       |
| 3       | Operações GraphQL      | Alternativa 1 | Média       |
| 4       | Padrões Avançados      | Alternativa 2 | Difícil     |
| 5       | Segurança              | Alternativa 4 | Média       |
| 6       | Paradigmas             | Alternativa 3 | Média       |
| 7       | Definição              | Alternativa 1 | Fácil       |
| 8       | Middlewares            | Alternativa 4 | Difícil     |
| 9       | Padrões Customizados   | Alternativa 3 | Difícil     |
| 10      | Decoradores            | Alternativa 4 | Difícil     |
| 11      | Tipagem                | Alternativa 3 | Média       |
