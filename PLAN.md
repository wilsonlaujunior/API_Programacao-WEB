# PLAN.md — Plano de Implementação Técnica (API de Gestão de Vendas e Entregas)

> **Papel Atual:** `[Backend Coder]`  
> **Fonte da Verdade:** `@[Módulo predefinido.openapi.json](Módulo%20predefinido.openapi.json)`  
> **Especificação Técnica:** `@[SPEC.md](SPEC.md)`  
> **Constituição e Diretrizes:** `@[STYLEGUIDE.md](STYLEGUIDE.md)`  
> **Governança:** `@[AGENTS.md](AGENTS.md)`  

---

## Índice das Fases de Implementação
1. [Fase 1: Ambiente, Dependências e Tratamento Semântico de Erros (RFC 9457)](#fase-1-ambiente-dependências-e-tratamento-semântico-de-erros-rfc-9457)
2. [Fase 2: Extração de Tipos e Contratos Pydantic v2 (Schema-First)](#fase-2-extração-de-tipos-e-contratos-pydantic-v2-schema-first)
3. [Fase 3: Camada de Persistência Relacional ORM (SQLAlchemy 2.0)](#fase-3-camada-de-persistência-relacional-orm-sqlalchemy-20)
4. [Fase 4: Blindagem de Segurança e Hash de Senhas (bcrypt)](#fase-4-blindagem-de-segurança-e-hash-de-senhas-bcrypt)
5. [Fase 5: Implementação dos 11 Endpoints HTTP (/v1/api)](#fase-5-implementação-dos-11-endpoints-http-v1api)
6. [Fase 6: Integração da Aplicação FastAPI e Inicialização de Dados](#fase-6-integração-da-aplicação-fastapi-e-inicialização-de-dados)
7. [Fase 7: Suíte de Testes Automatizados de Contrato (pytest)](#fase-7-suíte-de-testes-automatizados-de-contrato-pytest)

---

## Fase 1: Ambiente, Dependências e Tratamento Semântico de Erros (RFC 9457)

- [x] **1.1. Inicialização da Árvore de Diretórios do Projeto**
  - **Arquivo(s):** Criação dos diretórios `src/api/`, `src/api/routers/`, `src/api/tests/` e seus respectivos arquivos `__init__.py`.
  - **Critério de Aceite:** Estrutura modular idêntica à Seção 2 do `STYLEGUIDE.md`.

- [x] **1.2. Especificação Estrita de Dependências**
  - **Arquivo(s):** `src/api/requirements.txt`
  - **Critério de Aceite:** Conter as versões estáveis de:
    - `fastapi>=0.110.0`
    - `uvicorn[standard]>=0.28.0`
    - `pydantic>=2.6.0`
    - `pydantic-settings>=2.2.0`
    - `sqlalchemy>=2.0.28`
    - `bcrypt>=4.1.2`
    - `pytest>=8.1.0`
    - `httpx>=0.27.0`

- [x] **1.3. Implementação dos Modelos e Handlers de Erro RFC 9457**
  - **Arquivo(s):** `src/api/exceptions.py`
  - **Tarefas Granulares:**
    - Implementar a classe Pydantic `InvalidParam` (`name: str`, `reason: str`).
    - Implementar a classe Pydantic `ApiError` (`type: str`, `title: str`, `status: int`, `detail: str`, `instance: str`, `invalidParams: Optional[list[InvalidParam]]`).
    - Criar classes de exceção de domínio personalizadas (`DomainBadRequestException`, `DomainNotFoundException`, `DomainConflictException`).
    - Implementar `register_exception_handlers(app: FastAPI)` com captura de `RequestValidationError` (HTTP 422) e exceções de domínio retornando `media_type="application/problem+json"`.

---

## Fase 2: Extração de Tipos e Contratos Pydantic v2 (Schema-First)

- [x] **2.1. Enumerações e Tipos Básicos**
  - **Arquivo(s):** `src/api/schemas.py`
  - **Critério de Aceite:**
    - `StatusUsuario`: `"INATIVO"`, `"ATIVO"`
    - `StatusProduto`: `"ATIVO"`, `"INATIVO"`
    - `StatusFornecedor`: `"ATIVO"`, `"INATIVO"`
    - O campo `status_c` de cliente é `str` livre (sem enum no contrato atual).

- [x] **2.2. Schema Pydantic `Cliente`**
  - **Arquivo(s):** `src/api/schemas.py`
  - **Critério de Aceite:** Mapear `components.schemas.Cliente`:
    - `id_c`: UUID `str`
    - `nome_c`: `str`
    - `email_c`: `EmailStr`
    - `telefone_c`: `str`
    - `status_c`: `str`
    - Todos obrigatórios; suporte a `ConfigDict(populate_by_name=True, from_attributes=True)`.

- [x] **2.3. Schema Pydantic `Produto` e Schema de Cadastro**
  - **Arquivo(s):** `src/api/schemas.py`
  - **Critério de Aceite:**
    - Mapear `components.schemas.Produto`: `id_p` (UUID), `nome_p`, `descrição_p`, `preço_p` (float), `quantidadeEstoque` (int), `categoria`, `status_p` (`StatusProduto`), `fornecedorId` (UUID), `data_fabricação` (datetime), `data_validade` (datetime).
    - Criar `CreateProdutoDTO` para o body de `POST /v1/api/produtos` com os mesmos campos de negócio **sem exigir `id_p`** (conforme a Seção 2.3 do `SPEC.md`).

- [x] **2.4. Schema Pydantic `Fornecedor`**
  - **Arquivo(s):** `src/api/schemas.py`
  - **Critério de Aceite:** Mapear `components.schemas.Fornecedor`:
    - `id_f` (UUID), `razaoSocial_f` (3..150), `nomeFantasia_f` (3..100), `cnpj` (até 18), `email_f` (email), `telefone_f` (até 20), `endereco_f` (até 200), `cidade` (até 100), `estado` (até 2), `cep` (até 9), `status` (`StatusFornecedor`), `createdAT` (datetime), `modifiedAT` (datetime).

- [x] **2.5. Schema Pydantic `Usuario` e Schema Seguro de Saída**
  - **Arquivo(s):** `src/api/schemas.py`
  - **Critério de Aceite:**
    - Mapear `components.schemas.Usuario`: `id_u` (UUID), `nome_u` (3..100), `email_u` (email), `senha_u` (6..100, `writeOnly=True`), `telefone_u` (até 20), `status_u` (`StatusUsuario`), `createdAt_u` (datetime), `updateAt_u` (datetime).
    - Criar `UsuarioPublico` espelhando todos os campos **exceto `senha_u`**, para blindagem contra vazamento de credenciais em respostas.

---

## Fase 3: Camada de Persistência Relacional ORM (SQLAlchemy 2.0)

- [x] **3.1. Infraestrutura do Banco de Dados e Conexão**
  - **Arquivo(s):** `src/api/database.py`
  - **Critério de Aceite:**
    - `DATABASE_URL = "sqlite:///./vendas_entregas.db"`
    - Criação de `engine`, `SessionLocal` e dependência `get_db()`.
    - Declaração de `Base = DeclarativeBase`.

- [x] **3.2. Mapeamento das Entidades ORM**
  - **Arquivo(s):** `src/api/models.py`
  - **Critério de Aceite:**
    - **`UserModel` (tabela `usuarios`):** `id_u` (PK), `nome_u`, `email_u`, `senha_hash`, `telefone_u`, `status_u`, `created_at_u`, `update_at_u`.
    - **`ClientModel` (tabela `clientes`):** `id_c` (PK), `nome_c`, `email_c`, `telefone_c`, `status_c`.
    - **`SupplierModel` (tabela `fornecedores`):** `id_f` (PK), `razao_social_f`, `nome_fantasia_f`, `cnpj`, `email_f`, `telefone_f`, `endereco_f`, `cidade`, `estado`, `cep`, `status`, `created_at`, `modified_at`. Relacionamento 1:N com `produtos`.
    - **`ProductModel` (tabela `produtos`):** `id_p` (PK), `nome_p`, `descricao_p`, `preco_p`, `quantidade_estoque`, `categoria`, `status_p`, `fornecedor_id` (`ForeignKey("fornecedores.id_f")`), `data_fabricacao`, `data_validade`. Relacionamento N:1 com `fornecedores`.

---

## Fase 4: Blindagem de Segurança e Hash de Senhas (bcrypt)

- [x] **4.1. Hashing e Validação de Senhas**
  - **Arquivo(s):** `src/api/security.py`
  - **Critério de Aceite:**
    - Função `hash_password(plain_password: str) -> str` gerando salt aleatório seguro com `bcrypt`.
    - Função `verify_password(plain_password: str, hashed_password: str) -> bool`.
    - Garantir que a senha pura nunca seja persistida e nunca retorne em respostas da API.

---

## Fase 5: Implementação dos 11 Endpoints HTTP (/v1/api)

- [x] **5.1. Router de Clientes (`/v1/api/clientes`)**
  - **Arquivo(s):** `src/api/routers/clientes.py`
  - **Endpoints:**
    - `GET /v1/api/clientes`: Filtro opcional por query `nome_c`. Não exigir body na requisição (ajuste da pendência 2). Retorna lista de clientes.
    - `POST /v1/api/clientes`: Cadastra cliente usando schema `Cliente`.
    - `DELETE /v1/api/clientes/{id_c}`: Remove cliente pelo parâmetro de caminho `id_c`.

- [x] **5.2. Router de Produtos (`/v1/api/produtos`)**
  - **Arquivo(s):** `src/api/routers/produtos.py`
  - **Endpoints:**
    - `GET /v1/api/produtos/`: Rota com barra final publicada no OpenAPI. Query params opcionais: `nome_p` e `quantidadeEstoque`. Retorna `200 OK: list[Produto]`.
    - `POST /v1/api/produtos`: Body com `CreateProdutoDTO` (sem exigir `id_p`). Gera UUID para `id_p`. Valida integridade de `fornecedorId`.
    - `DELETE /v1/api/produtos/{id_p}`: Remove produto apontado por `id_p`.

- [x] **5.3. Router de Usuários (`/v1/api/usuarios`)**
  - **Arquivo(s):** `src/api/routers/usuarios.py`
  - **Endpoints:**
    - `GET /v1/api/usuarios`: Filtro opcional por query `nome_u`. Retorna lista de usuários sem expor `senha_u`.
    - `POST /v1/api/usuarios`: Cadastra usuário com schema `Usuario`. Hasheia `senha_u` antes de gravar no banco. Retorna representação segura sem expor senha.
    - `DELETE /v1/api/usuarios/{id_u}`: Remove usuário apontado por `id_u`.

- [x] **5.4. Router de Fornecedores (`/v1/api/fornecedores`)**
  - **Arquivo(s):** `src/api/routers/fornecedores.py`
  - **Endpoints:**
    - `GET /v1/api/fornecedores/{nome_f}`: Rota publicada com `nome_f` no path e `nome_f` opcional na query. Retorna fornecedor correspondente.
    - `POST /v1/api/fornecedores`: Cadastra fornecedor usando schema `Fornecedor`.
    - `DELETE /v1/api/fornecedores/{id_f}`: Remove fornecedor por `id_f`.

---

## Fase 6: Integração da Aplicação FastAPI e Inicialização de Dados

- [x] **6.1. Dados Iniciais de Demonstração (Fixtures)**
  - **Arquivo(s):** `src/api/database.py`
  - **Critério de Aceite:**
    - Função idempotente `init_db()` que cria as tabelas do SQLAlchemy.
    - Rotina de seed para popular registros básicos de teste sem colidir com regras de negócio.

- [x] **6.2. Aplicação Principal FastAPI e Configuração de Rotas**
  - **Arquivo(s):** `src/api/main.py`
  - **Critério de Aceite:**
    - Instância FastAPI com lifespan executando `init_db()`.
    - Configuração de `CORSMiddleware`.
    - Registro dos handlers globais de erro RFC 9457 (`application/problem+json`).
    - Inclusão dos 4 routers com prefixo `/v1/api` correspondendo aos 11 endpoints publicados.

---

## Fase 7: Suíte de Testes Automatizados de Contrato (pytest)

- [x] **7.1. Fixtures de Teste com SQLite em Memória**
  - **Arquivo(s):** `src/api/tests/conftest.py`
  - **Critério de Aceite:** Fixture `client` utilizando `TestClient` sobre banco isolado em memória (`sqlite:///:memory:`).

- [x] **7.2. Testes de Contrato da Coleção Clientes**
  - **Arquivo(s):** `src/api/tests/test_clientes.py`
  - **Cenários Cobertos:** `GET /v1/api/clientes` (com/sem filtro `nome_c`), `POST /v1/api/clientes`, `DELETE /v1/api/clientes/{id_c}`.

- [x] **7.3. Testes de Contrato da Coleção Produtos**
  - **Arquivo(s):** `src/api/tests/test_produtos.py`
  - **Cenários Cobertos:** `GET /v1/api/produtos/` (com/sem filtros `nome_p` e `quantidadeEstoque`), `POST /v1/api/produtos` (sem exigir `id_p` no body), vínculo de `fornecedorId`, `DELETE /v1/api/produtos/{id_p}`.

- [x] **7.4. Testes de Contrato da Coleção Usuários**
  - **Arquivo(s):** `src/api/tests/test_usuarios.py`
  - **Cenários Cobertos:** `GET /v1/api/usuarios`, `POST /v1/api/usuarios`, blindagem total contra retorno da `senha_u`, `DELETE /v1/api/usuarios/{id_u}`.

- [x] **7.5. Testes de Contrato da Coleção Fornecedores**
  - **Arquivo(s):** `src/api/tests/test_fornecedores.py`
  - **Cenários Cobertos:** `GET /v1/api/fornecedores/{nome_f}`, `POST /v1/api/fornecedores`, `DELETE /v1/api/fornecedores/{id_f}`.

- [x] **7.6. Testes de Tratamento Semântico de Erros (RFC 9457)**
  - **Arquivo(s):** `src/api/tests/test_errors.py`
  - **Cenários Cobertos:** Validação de payloads inválidos retornando `422 Unprocessable Content` com media type `application/problem+json` e schema `ApiError`.

- [x] **7.7. Execução e Homologação da Suíte Completa**
  - Execução no terminal: `pytest -v --tb=short src/api/tests/`
  - **Critério de Conclusão:** 100% dos testes aprovados cobrindo os 11 métodos publicados.
