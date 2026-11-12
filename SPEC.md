# SPEC.md — Especificação de Negócio e Contrato (API de Gestão de Vendas e Entregas)

## 1. Visão Geral e Fonte da Verdade do Contrato

Este projeto desenvolve uma **API REST de Gestão de Vendas e Entregas**, destinada a organizar o cadastro de fornecedores, produtos, usuários e clientes/destinatários, com evolução planejada para gerenciar vendas, tarefas de entrega, responsáveis, prazos e acompanhamento da execução.

**Equipe do Projeto (Fatec):**
- Adriana Gasparetto Martelli (RA: 2840482423026)
- Alexandre Victoriano Ribeiro Ulhoa (RA: 2840482423007)
- Wilson Lau Junior (RA: 2840482423013)

**Objetivo funcional informado:** permitir que uma operação de venda se relacione com produtos e seus fornecedores, com um usuário responsável e com o cliente/destinatário, de forma que entregas tenham tarefas e datas-limite de conclusão.

O contrato formal **atualmente disponível** (rotas, parâmetros, exemplos, respostas e JSON Schemas) está no arquivo [`Módulo predefinido.openapi.json`](Módulo%20predefinido.openapi.json), exportado do **Apidog**, em OpenAPI 3.0.1. O arquivo exportado apresenta o título interno `Módulo predefinido`, versão `1.0.0`.

**Hierarquia e governança:**
1. **Apidog/OpenAPI:** fonte primária do **contrato publicado**. Não presumir endpoints, parâmetros, exemplos ou validações inexistentes.
2. **Este `SPEC.md`:** define o escopo de negócio e descreve o contrato atual, distinguindo-o de decisões **propostas** ou **pendentes de aprovação no Apidog**.
3. **`STYLEGUIDE.md`:** define a arquitetura técnica e as convenções de implementação, incluindo a proposta de respostas HTTP REST e o padrão RFC 9457.
4. **`AGENTS.md` e `PLAN.md`:** disciplinam as personas, a sequência de trabalho e o acompanhamento das etapas. **Não modificar código fora de uma etapa aprovada em `PLAN.md`.**

Em caso de divergência, **registrar o conflito e sincronizar o contrato no Apidog antes de codificar o comportamento novo**. A especificação não autoriza alterações implícitas no contrato.

### 1.1. Escopo do contrato atual

Recursos definidos no OpenAPI: **Cliente, Produto, Usuario e Fornecedor**. As operações disponíveis são listagem/consulta, cadastro e exclusão. O prefixo efetivamente publicado é **`/v1/api`** (e **não** `/api/v1`).

**Fora do contrato atual:** Pedido/Venda, ItemVenda, TarefaEntrega, Entrega, Destinatário como recurso independente, prazo de entrega, status de entrega, comprovante, autenticação, paginação, endpoints `PATCH`/`PUT`, rotas de consulta individual por ID e endpoints de atualização. Esses itens só poderão ser exigidos na implementação depois de definidos no Apidog.

---

## 2. Prioridade Arquitetural: Extração de Tipos (Schema-First)

Antes de implementar persistência ou endpoints, mapear os `components.schemas` do OpenAPI para **Pydantic v2** em `src/api/schemas.py`, preservando **exatamente** os nomes externos dos atributos, tipos, formatos, obrigatoriedades e limites realmente presentes no contrato.

A pilha e a estrutura de pastas seguem o `STYLEGUIDE.md`: Python 3.12+, FastAPI, Pydantic v2, persistência isolada em `database.py` e testes com `pytest`/`TestClient`.

### 2.1. Enumerações existentes no contrato

- **Status de usuário (`status_u`):** `"INATIVO"`, `"ATIVO"`.
- **Status de produto (`status_p`):** `"ATIVO"`, `"INATIVO"`.
- **Status de fornecedor (`status`):** `"ATIVO"`, `"INATIVO"`.
- **Status de cliente (`status_c`):** `string` livre; **não há enum definido**.

As classes Enum internas podem ser reutilizadas quando tiverem os mesmos valores, mas os campos JSON devem manter seus nomes originais. **Não inventar enum de venda, pedido ou entrega nesta fase.**

### 2.2. Schema `Usuario`

| Campo JSON | Tipo/formato OpenAPI | Restrições publicadas |
|---|---|---|
| `id_u` | `string`, `uuid` | Obrigatório |
| `nome_u` | `string` | Obrigatório; 3 a 100 caracteres |
| `email_u` | `string`, `email` | Obrigatório |
| `senha_u` | `string` | Obrigatório; 6 a 100 caracteres |
| `telefone_u` | `string` | Obrigatório; até 20 caracteres |
| `status_u` | `string`, enum | Obrigatório; `INATIVO` ou `ATIVO` |
| `createdAt_u` | `string`, `date-time` | Obrigatório |
| `updateAt_u` | `string`, `date-time` | Obrigatório |

O `POST /v1/api/usuarios` referencia esse schema integralmente. **Pendência de segurança:** não devolver `senha_u` em respostas e armazenar apenas hash seguro da senha; propor no Apidog schemas separados de entrada e saída. O OpenAPI atual **não especifica** autenticação nem proteção de rotas.

### 2.3. Schema `Produto`

| Campo JSON | Tipo/formato OpenAPI | Restrições publicadas |
|---|---|---|
| `id_p` | `string`, `uuid` | Obrigatório no schema `Produto` |
| `nome_p` | `string` | Obrigatório |
| `descrição_p` | `string` | Obrigatório |
| `preço_p` | `number` | Obrigatório |
| `quantidadeEstoque` | `integer` | Obrigatório |
| `categoria` | `string` | Obrigatório |
| `status_p` | `string`, enum | Obrigatório; `ATIVO` ou `INATIVO` |
| `fornecedorId` | `string`, `uuid` | Obrigatório |
| `data_fabricação` | `string`, `date-time` | Obrigatório |
| `data_validade` | `string`, `date-time` | Obrigatório |

**Atenção:** `POST /v1/api/produtos` **não referencia** `#/components/schemas/Produto`; utiliza um schema inline com os mesmos campos de negócio **exceto `id_p`**, o que significa que o corpo de cadastro do produto **não exige** `id_p` no contrato atual. O exemplo fornecido nesse POST está incompleto em relação aos próprios campos obrigatórios e usa `fornecedorId: "13"`, que **não é UUID válido**; corrigir no Apidog antes de utilizar o exemplo como teste positivo.

Os limites de estoque, a não negatividade do preço e a regra entre datas de fabricação e validade **não estão especificados** como validações no contrato: são sugestões de evolução, não requisitos já publicados.

### 2.4. Schema `Cliente`

| Campo JSON | Tipo/formato OpenAPI | Restrições publicadas |
|---|---|---|
| `id_c` | `string`, `uuid` | Obrigatório |
| `nome_c` | `string` | Obrigatório |
| `email_c` | `string`, `email` | Obrigatório |
| `telefone_c` | `string` | Obrigatório |
| `status_c` | `string` | Obrigatório; enum não definido |

O cliente poderá exercer o papel de **destinatário** no fluxo de entregas quando o módulo correspondente for contratado. O esquema atual **não contém endereço de entrega**.

### 2.5. Schema `Fornecedor`

| Campo JSON | Tipo/formato OpenAPI | Restrições publicadas |
|---|---|---|
| `id_f` | `string`, `uuid` | Obrigatório |
| `razaoSocial_f` | `string` | Obrigatório; 3 a 150 caracteres |
| `nomeFantasia_f` | `string` | Obrigatório; 3 a 100 caracteres |
| `cnpj` | `string` | Obrigatório; até 18 caracteres |
| `email_f` | `string`, `email` | Obrigatório |
| `telefone_f` | `string` | Obrigatório; até 20 caracteres |
| `endereco_f` | `string` | Obrigatório; até 200 caracteres |
| `cidade` | `string` | Obrigatório; até 100 caracteres |
| `estado` | `string` | Obrigatório; até 2 caracteres |
| `cep` | `string` | Obrigatório; até 9 caracteres |
| `status` | `string`, enum | Obrigatório; `ATIVO` ou `INATIVO` |
| `createdAT` | `string`, `date-time` | Obrigatório |
| `modifiedAT` | `string`, `date-time` | Obrigatório |

O schema **não especifica** validação de dígitos do CNPJ, máscara exata de CEP ou unicidade do cadastro; decisões de negócio futuras devem ser aprovadas no contrato.

### 2.6. DTOs, respostas e erros

- **Schemas publicados:** apenas `Usuario`, `Produto`, `Cliente` e `Fornecedor`.
- **DTOs de criação separados:** ainda não existem como `components.schemas`. As rotas POST de cliente, usuário e fornecedor usam o schema integral (incluindo IDs e timestamps); o POST de produto usa schema inline distinto.
- **Atualização:** não existem `Update*DTO` nem endpoints PATCH/PUT.
- **Listagens:** somente `GET /v1/api/produtos/` possui resposta `200` modelada como `array` de `Produto`. Os demais GETs possuem objeto vazio como schema de resposta (`type: object`, `properties: {}`): o envelope de listagem precisa ser definido no Apidog.
- **ApiError (proposta técnica, ainda não no OpenAPI):** `type` (URI/string), `title` (string), `status` (integer), `detail` (string), `instance` (string) e `invalidParams` (lista de `{name, reason}` para falhas de validação), de acordo com RFC 9457 e seção 3.1 do Styleguide.

---

## 3. Regras de Negócio e Comportamento dos Endpoints

**Nota sobre status HTTP:** o contrato exportado descreve **`200 OK` para todas as operações**, inclusive POST e DELETE. O Styleguide propõe **`201 Created`** para cadastros e **`204 No Content`** para exclusões. A adoção desses códigos depende de **atualização prévia no Apidog**. Os textos abaixo distinguem contrato vigente de comportamento-alvo.

### 3.1. Coleção `Clientes` (`/v1/api/clientes`)

- **`GET /v1/api/clientes`**
  - Consulta/listagem de clientes.
  - Query opcional `nome_c` (`string`); sem paginação definida.
  - **Contrato atual:** `200 OK`, schema de resposta vazio e `requestBody` JSON vazio marcado como obrigatório. **Ajuste proposto:** remover o corpo obrigatório do GET e definir um schema de listagem.
- **`POST /v1/api/clientes`**
  - Cadastra cliente usando `#/components/schemas/Cliente` (todos os cinco campos obrigatórios, inclusive `id_c`).
  - **Contrato atual:** `200 OK`, resposta de objeto vazio.
  - **Comportamento-alvo:** `201 Created`, com resposta tipada e cabeçalho `Location` quando existir rota individual de consulta.
- **`DELETE /v1/api/clientes/{id_c}`**
  - Remove cliente identificado pelo parâmetro de caminho `id_c` (`string` sem `format: uuid` na definição do parâmetro).
  - **Contrato atual:** `200 OK`, corpo de resposta com schema vazio.
  - **Comportamento-alvo:** `204 No Content`, sem corpo; `404` para inexistência depois de documentado.

### 3.2. Coleção `Produtos` (`/v1/api/produtos`)

- **`GET /v1/api/produtos/`**
  - Lista produtos; query opcional `nome_p` (`string`) e `quantidadeEstoque` (`integer`).
  - **Contrato atual:** `200 OK`, `array` de `Produto`; `requestBody` JSON vazio obrigatório indevidamente.
  - **Ajuste proposto:** remover corpo de GET e padronizar barra final com as outras rotas, após atualização do Apidog.
- **`POST /v1/api/produtos`**
  - Cadastra produto via schema inline; campos obrigatórios: `nome_p`, `descrição_p`, `preço_p`, `quantidadeEstoque`, `categoria`, `status_p`, `fornecedorId`, `data_fabricação`, `data_validade`. **Não exigir `id_p` no body** porque ele não é requerido nesta operação.
  - O campo `fornecedorId` tem tipo `string`/`uuid` e representa o vínculo pretendido ao fornecedor.
  - **Contrato atual:** `200 OK`, schema de resposta vazio.
  - **Comportamento-alvo:** validar existência do fornecedor (regra ainda não descrita no contrato), retornar `201 Created` e apresentar recurso criado sem divergência de schema.
- **`DELETE /v1/api/produtos/{id_p}`**
  - Remove produto indicado por `id_p` (`string` no path, sem formato UUID declarado).
  - **Contrato atual:** `200 OK` e `requestBody` JSON vazio obrigatório indevidamente.
  - **Comportamento-alvo:** `204 No Content` e requisição sem corpo; erros semânticos após documentação.

### 3.3. Coleção `Usuários` (`/v1/api/usuarios`)

- **`GET /v1/api/usuarios`**
  - Consulta/listagem; query opcional `nome_u` (`string`).
  - **Contrato atual:** `200 OK`, schema de resposta vazio, GET com corpo JSON vazio obrigatório indevidamente.
  - **Ajustes propostos:** definir formato da listagem, eliminar body e **garantir omissão de senha** em toda resposta.
- **`POST /v1/api/usuarios`**
  - Cadastra usuário com schema `Usuario`, atualmente incluindo `id_u`, datas e `senha_u` como obrigatórios.
  - **Contrato atual:** `200 OK`, resposta de objeto vazio.
  - **Comportamento-alvo:** `201 Created`, hash da senha na persistência e schema seguro de saída sem senha, após revisão contratual.
- **`DELETE /v1/api/usuarios/{id_u}`**
  - Remove usuário apontado por `id_u` (`string` sem `format: uuid` no parâmetro).
  - **Contrato atual:** `200 OK`.
  - **Comportamento-alvo:** `204 No Content`, sem corpo.

### 3.4. Coleção `Fornecedores` (`/v1/api/fornecedores`)

- **`GET /v1/api/fornecedores/{nome_f}`**
  - Consulta pelo caminho `nome_f` (`string`, obrigatório), com **outro parâmetro também chamado `nome_f` em query**, este opcional.
  - **Contrato atual:** `200 OK`, schema de resposta vazio e `requestBody` JSON vazio obrigatório indevidamente.
  - **Pendência:** esclarecer se a intenção é listar fornecedores com filtro (`GET /v1/api/fornecedores?nome_f=...`) ou buscar um fornecedor por nome. A rota atual não é listagem geral. Evitar manter o mesmo filtro simultaneamente no path e na query sem necessidade.
- **`POST /v1/api/fornecedores`**
  - Cadastra fornecedor via schema `Fornecedor` completo (incluindo `id_f`, `createdAT` e `modifiedAT`).
  - **Contrato atual:** `200 OK`, resposta vazia.
  - **Comportamento-alvo:** `201 Created` e representação tipada do recurso.
- **`DELETE /v1/api/fornecedores/{id_f}`**
  - Remove fornecedor por `id_f` (`string`, sem `format: uuid` declarado no path).
  - **Contrato atual:** `200 OK`.
  - **Comportamento-alvo:** `204 No Content`, sem corpo; validar vínculos com produtos somente após definição da política de exclusão no contrato.

### 3.5. Relacionamentos e integridade dos dados

- **Fornecedor → Produto:** o único relacionamento explícito no OpenAPI é `Produto.fornecedorId` (UUID). **Regra de negócio proposta:** cada produto referencia um fornecedor cadastrado; ao excluir fornecedor, estabelecer uma política explícita para produtos vinculados (bloqueio, transferência, inativação ou cascata). Não presumir qual política vigora.
- **Cliente/destinatário:** o schema Cliente identifica pessoas/organizações que poderão receber entregas, mas **nenhum vínculo de cliente com venda ou entrega está especificado**.
- **Usuário:** cadastrado como recurso independente; **não existe ainda atributo que o associe a pedido, tarefa ou entrega**.
- **Venda, itens e tarefas:** relacionamentos, cardinalidades, identificadores e regras de prazo **a definir** em fase posterior.

---

## 4. Expectations do Apidog, Resiliência e Qualidade

### 4.1. Contrato publicado versus testes futuros

O arquivo OpenAPI enviado **não define** UUIDs especiais para simular `404`, não apresenta seeds de exemplo confiáveis para a base e não documenta respostas HTTP `400`, `404`, `409` ou `422`, nem um componente `ApiError`. **Não copiar** os UUIDs da TaskFlow API, seu domínio de erro ou dados de seed do modelo de referência.

### 4.2. Tratamento de erros a formalizar

Após inserir as respostas e o schema `ApiError` no Apidog:

- **400 Bad Request:** JSON sintaticamente inválido e demais erros de requisição que o contrato atribuir a 400.
- **404 Not Found:** recurso inexistente em operações individuais, após documentação de cada cenário.
- **409 Conflict:** violação de integridade, duplicidade e outras regras que forem formalizadas.
- **422 Unprocessable Content:** violação de campos, tipos, formatos ou validações semânticas definidas no contrato.
- **Formato:** RFC 9457 com `type`, `title`, `status`, `detail`, `instance` e `invalidParams` quando aplicável.

O middleware de JSON malformado deverá seguir o `STYLEGUIDE.md`, mas suas respostas de erro devem ser documentadas primeiro no Apidog.

### 4.3. Testes de contrato

A suíte automatizada com `pytest` e `TestClient` deve cobrir, conforme o contrato revisado:

1. Operações GET, POST e DELETE existentes para os quatro recursos.
2. Tipos, campos obrigatórios, `minLength`, `maxLength`, `format: email`, `format: uuid`, `format: date-time` e enums publicados.
3. Filtros `nome_c`, `nome_p`, `quantidadeEstoque`, `nome_u` e `nome_f` na localização exata (query/path) publicada.
4. Ausência de senha em respostas de usuário, após criação de DTO seguro.
5. Relação `fornecedorId`/fornecedor e decisões de integridade depois de formalizadas.
6. Status HTTP e respostas de erro **exatamente como estiverem na versão sincronizada do Apidog**.
7. JSON malformado, 404 e validação 422 após suas Expectations serem definidas.

Não exigir aprovação de testes de funcionalidades ainda não descritas no contrato.

### 4.4. Dados iniciais

**Não há seed oficial no OpenAPI.** Se for necessário criar registros de demonstração para testes de integração, usar UUIDs de teste próprios, identificá-los como fixtures e isolar/resetar o estado entre testes. Não tratá-los como dados oficiais do Apidog.

---

## 5. Evolução Planejada: Vendas e Entregas (não contratada)

A visão de negócio inclui uma **lista de tarefas para realização de entregas**, com **data-limite** e vinculação entre **fornecedor, produto, usuário responsável e cliente/destinatário**. Para completar o objetivo semestral, é necessário especificar no Apidog, em uma etapa futura, as seguintes decisões:

- **Venda/Pedido:** identificador, cliente, usuário responsável, itens, quantidades, valores, data e estado de processamento.
- **Itens de venda:** produto, quantidade e preço registrado no momento da venda; definição do vínculo com o fornecedor por meio do produto.
- **Tarefa/Entrega:** identificador, pedido ou venda vinculada, responsável, destinatário, data-limite, status e data de conclusão.
- **Endereço do destinatário:** origem dos dados e tratamento de múltiplos endereços.
- **Fluxo de status:** estados possíveis de pedido e entrega; transições permitidas, atraso, cancelamento e conclusão.
- **Regras de negócio:** consistência de estoque, impedimento de entrega sem pedido válido, exclusões com relacionamentos e restrições de datas.
- **Rotas, DTOs e respostas:** especificar todos os métodos, caminhos, parâmetros, schemas, exemplos, status e Expectations antes de implementar.

**Nenhum dos nomes de campos, enums ou rotas futuros acima constitui contrato vigente.** Ao definir essas estruturas, atualizar nesta ordem: **Apidog → SPEC.md → STYLEGUIDE.md (quando necessário) → PLAN.md → código/testes**.

---

## 6. Pendências Objetivas para Sincronização Design-First

| Nº | Situação encontrada no OpenAPI | Ajuste a decidir e registrar no Apidog |
|---|---|---|
| 1 | Todos os POST e DELETE retornam `200` | Avaliar `201` para criação e `204` sem corpo para exclusão, conforme Styleguide |
| 2 | GETs e DELETE de produto exigem corpo JSON vazio | Remover `requestBody` dessas operações |
| 3 | GET de produtos usa `/v1/api/produtos/`, POST usa `/v1/api/produtos` | Padronizar rota sem/sob barra final |
| 4 | Fornecedor usa `nome_f` obrigatório no path e opcional na query | Definir consulta individual e/ou listagem filtrável |
| 5 | Schema de saída de 3 GETs e todos os POST/DELETEs é objeto vazio | Criar respostas reutilizáveis tipadas |
| 6 | POST de Cliente, Usuário e Fornecedor exige ID e timestamps | Definir DTOs próprios com campos gerados pelo servidor, caso esta seja a regra desejada |
| 7 | POST de Produto possui schema inline diferente de `Produto` | Criar `CreateProdutoDTO` reutilizável; preservar a ausência de `id_p` no body |
| 8 | Exemplo do POST de Produto é inválido/incompleto | Fornecer exemplo válido com UUID e datas |
| 9 | `Usuario` contém senha em schema potencialmente reutilizável na saída | Criar DTO seguro sem `senha_u`; armazenar hash de senha |
| 10 | Schemas de erro e Expectations de 400/404/409/422 inexistentes | Adicionar `ApiError`, exemplos e cenários documentados |
| 11 | `securitySchemes` vazio e nenhum endpoint declara autenticação | Decidir autenticação/autorização e registrar no contrato, se exigido |
| 12 | IDs de path são `string` sem formato UUID, embora IDs dos schemas sejam UUID | Uniformizar os formatos dos parâmetros, se desejado |
| 13 | Campos como `descrição_p`, `preço_p`, `createdAT`, `updateAt_u` usam estilos distintos | Decidir padronização; não renomear sem atualização do Apidog |
| 14 | Pedido/Venda, TarefaEntrega e prazo de entrega inexistentes | Especificar novos recursos para cobrir o objetivo do projeto |

---

## 7. Critérios de Aceite para a Primeira Implementação

A primeira fase poderá ser considerada concluída quando:

- Os quatro schemas atuais estiverem representados em Pydantic v2, com aliases JSON corretos.
- Os **11 métodos HTTP** já definidos no Apidog estiverem implementados e testados.
- Os contratos de request/response e respectivos status tiverem sido reconciliados com o Apidog **antes** de utilizar comportamentos-alvo novos.
- O vínculo `fornecedorId` estiver tecnicamente representado, com regras de integridade formalizadas se forem implementadas.
- Os dados sensíveis de usuário não forem retornados indevidamente; a alteração contratual necessária estiver registrada.
- Os testes cobrirem as restrições publicadas, sem pressupor regras ausentes.
- A pessoa responsável pelo planejamento atualizar o `PLAN.md` e a implementação respeitar as personas do `AGENTS.md`.

> **Estado desta especificação:** versão inicial baseada no OpenAPI recebido em 28/10/2026. Itens explicitamente marcados como *propostos*, *pendentes* ou *futuros* não são endpoints nem requisitos já publicados no Apidog.
