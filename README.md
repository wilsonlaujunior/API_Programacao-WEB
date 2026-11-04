# API de Gestão de Vendas e Entregas

## 1. Visão geral

A **API de Gestão de Vendas e Entregas** é uma API REST acadêmica, especificada segundo a abordagem **Design-First**. Seu contrato define operações para organizar os cadastros de clientes, produtos, usuários e fornecedores de um pequeno negócio.

A especificação é documentada no **Apidog**, utilizando o padrão **OpenAPI 3.0.1**. As requisições e respostas utilizam o formato JSON e os recursos são acessados por endpoints versionados.

## 2. Objetivo e público-alvo

O objetivo do projeto é disponibilizar uma interface padronizada para cadastrar, consultar, atualizar e remover informações essenciais à organização de clientes, produtos e fornecedores, além de manter os registros de usuários do sistema.

O público-alvo são **pequenos negócios, equipes administrativas e operadores responsáveis pelo controle de cadastros e estoque**, que necessitam centralizar e consultar essas informações por meio de uma API.

## 3. Recursos e funcionalidades

A versão atual do contrato contempla quatro recursos:

- **Produtos:** cadastro, listagem com filtros, consulta por identificador, atualização parcial e remoção. Inclui informações de preço, estoque, categoria, situação, datas e fornecedor relacionado.
- **Fornecedores:** cadastro, listagem com filtros, consulta por identificador, atualização parcial e remoção. Inclui dados cadastrais e informações de contato.
- **Clientes:** cadastro, listagem com filtro por nome e remoção por identificador.
- **Usuários:** cadastro, listagem com filtro por nome e remoção por identificador. As respostas públicas não expõem a senha.

**Escopo do CRUD:** Produtos e Fornecedores possuem o ciclo CRUD completo. Clientes e Usuários possuem, nesta versão, operações de criação, listagem e remoção; consulta individual e atualização desses dois recursos não estão previstas no contrato atual.

## 4. Arquitetura dos recursos e relacionamentos

A API organiza suas operações por recurso, utilizando endpoints próprios e schemas reutilizáveis definidos em `components/schemas` na especificação OpenAPI.

### 4.1. Produtos

Representa os itens cadastrados para controle. Seus principais campos são:

- `id_p`: identificador do produto;
- `nome_p` e `descrição_p`: nome e descrição;
- `preço_p`: preço;
- `quantidadeEstoque`: quantidade disponível;
- `categoria` e `status_p`: classificação e situação;
- `fornecedorId`: identificador do fornecedor relacionado;
- `data_fabricação` e `data_validade`: datas do produto.

### 4.2. Fornecedores

Representa as empresas ou pessoas que fornecem os produtos. Seus campos incluem `id_f`, `razaoSocial_f`, `nomeFantasia_f`, `cnpj`, `email_f`, `telefone_f`, `endereco_f`, `cidade`, `estado`, `cep`, `status`, `createdAT` e `modifiedAT`.

### 4.3. Clientes

Representa as pessoas cadastradas como clientes. Seus campos incluem `id_c`, `nome_c`, `email_c`, `telefone_c` e `status_c`.

### 4.4. Usuários

Representa as pessoas autorizadas a utilizar o sistema. Seus campos incluem `id_u`, `nome_u`, `email_u`, `senha_u`, `telefone_u`, `status_u`, `createdAt_u` e `updateAt_u`.

A senha é um dado de entrada e não deve ser apresentada nas respostas públicas da API.

### 4.5. Relacionamento entre recursos

O campo `fornecedorId`, presente no recurso Produto, identifica o fornecedor associado ao produto. Os demais relacionamentos de vendas, pedidos e entregas ainda não fazem parte do contrato desta versão.

## 5. Endpoints documentados

A versão atual utiliza o prefixo **`/api/v1`**.

| Recurso | Método | Endpoint | Finalidade |
|---|---|---|---|
| Produtos | GET | `/api/v1/produtos` | Listar e filtrar produtos |
| Produtos | POST | `/api/v1/produtos` | Cadastrar produto |
| Produtos | GET | `/api/v1/produtos/{id_p}` | Consultar produto por ID |
| Produtos | PATCH | `/api/v1/produtos/{id_p}` | Atualizar parcialmente um produto |
| Produtos | DELETE | `/api/v1/produtos/{id_p}` | Remover produto |
| Fornecedores | GET | `/api/v1/fornecedores` | Listar e filtrar fornecedores |
| Fornecedores | POST | `/api/v1/fornecedores` | Cadastrar fornecedor |
| Fornecedores | GET | `/api/v1/fornecedores/{id_f}` | Consultar fornecedor por ID |
| Fornecedores | PATCH | `/api/v1/fornecedores/{id_f}` | Atualizar parcialmente um fornecedor |
| Fornecedores | DELETE | `/api/v1/fornecedores/{id_f}` | Remover fornecedor |
| Clientes | GET | `/api/v1/clientes` | Listar e filtrar clientes |
| Clientes | POST | `/api/v1/clientes` | Cadastrar cliente |
| Clientes | DELETE | `/api/v1/clientes/{id_c}` | Remover cliente |
| Usuários | GET | `/api/v1/usuarios` | Listar e filtrar usuários |
| Usuários | POST | `/api/v1/usuarios` | Cadastrar usuário |
| Usuários | DELETE | `/api/v1/usuarios/{id_u}` | Remover usuário |

### 5.1. Parâmetros de consulta

As operações de listagem aceitam parâmetros para filtragem e paginação:

- **Produtos:** `nome_p`, `quantidadeEstoque`, `status_p`, `page` e `pageSize`.
- **Fornecedores:** `razaoSocial_f`, `nomeFantasia_f`, `cnpj`, `status`, `page` e `pageSize`.
- **Clientes:** `nome_c`, `page` e `pageSize`.
- **Usuários:** `nome_u`, `page` e `pageSize`.

Os parâmetros opcionais, seus tipos, limites e valores permitidos estão detalhados na especificação OpenAPI do Apidog.

## 6. Padrões técnicos e documentação

- **Estilo arquitetural:** REST.
- **Abordagem de desenvolvimento:** Design-First.
- **Formato de dados:** JSON.
- **Protocolo:** HTTP/HTTPS.
- **Prefixo de versionamento das rotas:** `/api/v1`.
- **Ferramenta de design e documentação:** Apidog.
- **Formato da especificação:** OpenAPI 3.0.1.
- **Identificadores:** UUID nos recursos em que esse formato está definido no contrato.
- **Schemas:** modelos de dados e DTOs definidos em `components/schemas` e referenciados pelas operações.

A especificação documenta respostas de sucesso e de erro para as operações, incluindo validações de entrada e situações como recurso não encontrado ou conflito de integridade, conforme cada endpoint.

A implementação prevista para o projeto utiliza **Python 3.12 ou superior, FastAPI, Pydantic v2 e pytest**. Essas tecnologias compõem a stack planejada para implementação e testes; este README descreve principalmente o contrato da API.

## 7. Equipe

| Integrante | Registro acadêmico |
|---|---|
| Adriana Gasparetto Martelli | 2840482423026 |
| Alexandre Vitoriano Ulloa | 2840482423*** |
| Wilson Lau Junior | 2840482423013 |

## 8. Limites do escopo e possibilidades de evolução

Apesar do nome do projeto, o contrato atual está concentrado no gerenciamento dos cadastros de clientes, produtos, usuários e fornecedores. **Não estão documentadas nesta versão operações de vendas, pedidos, itens de venda ou acompanhamento de entregas.**

Como evolução futura, o projeto poderá incorporar recursos para registrar pedidos e vendas, relacionar itens vendidos, organizar tarefas e responsáveis por entregas, controlar prazos e status e registrar comprovantes de recebimento. Cada novo recurso deverá ser previamente especificado e documentado no Apidog antes de integrar o contrato da API.
