# STYLEGUIDE.md — Constituição Técnica (API de Gestão de Vendas e Entregas)

## 1. Stack Tecnológica e Versões Obrigatórias
- **Linguagem:** Python 3.12+ com tipagem estática rigorosa (`typing`, `UUID`, `datetime`, `Enum`, `Annotated`).
- **Framework Web:** FastAPI (com suporte assíncrono e roteamento modular via `APIRouter`).
- **Validação e Serialização:** Pydantic v2 (`BaseModel`, `ConfigDict(populate_by_name=True, from_attributes=True)`, `Field`, `@field_validator`).
- **Persistência e ORM:** SQLAlchemy 2.0+ utilizando sintaxe declarativa moderna (`DeclarativeBase`, `Mapped`, `mapped_column`, `relationship`, `ForeignKey`) com SQLite (`sqlite:///./vendas_entregas.db` para runtime e `sqlite:///:memory:` para testes).
- **Tratamento de Senhas:** Hash seguro com `bcrypt` (a senha informada em `senha_u` deve ser protegida antes de persistida e nunca exposta em respostas).
- **Suíte de Testes Automatizados:** `pytest` + `httpx` + `fastapi.testclient.TestClient`.

---

## 2. Arquitetura de Diretórios e Separação de Camadas
Todo o código da API deve respeitar rigorosamente a estrutura modular abaixo:

```text
src/api/
├── __init__.py
├── main.py          # Instância FastAPI, lifespan (init_db + seed), CORS, handlers de erro e inclusão de routers
├── schemas.py       # Contratos Pydantic v2 mapeando os components.schemas do OpenAPI (Usuario, Produto, Cliente, Fornecedor)
├── models.py        # Entidades ORM SQLAlchemy 2.0 (UserModel, ProductModel, ClientModel, SupplierModel)
├── database.py      # Engine SQLAlchemy, SessionLocal, get_db (Dependency Injection) e dados de inicialização
├── security.py      # Utilitários de hash bcrypt e segurança de senhas
├── exceptions.py    # Exceções de domínio e handlers padronizados no formato RFC 9457 (Problem Details)
├── requirements.txt # Dependências estritas do projeto
├── routers/
│   ├── __init__.py
│   ├── clientes.py      # Endpoints da coleção /v1/api/clientes (GET, POST, DELETE)
│   ├── produtos.py      # Endpoints da coleção /v1/api/produtos (GET /v1/api/produtos/, POST, DELETE)
│   ├── usuarios.py      # Endpoints da coleção /v1/api/usuarios (GET, POST, DELETE)
│   └── fornecedores.py  # Endpoints da coleção /v1/api/fornecedores (GET /{nome_f}, POST, DELETE /{id_f})
└── tests/
    ├── __init__.py
    ├── conftest.py          # Fixtures de banco SQLite em memória e TestClient
    ├── test_clientes.py     # Testes de contrato das operações de clientes
    ├── test_produtos.py     # Testes de contrato das operações de produtos e vínculo fornecedorId
    ├── test_usuarios.py     # Testes de contrato de usuários e proteção de senha
    ├── test_fornecedores.py # Testes de contrato das operações de fornecedores
    └── test_errors.py       # Testes de conformidade com tratamento semântico de erros (RFC 9457)
```

---

## 3. Regras Inegociáveis de Implementação

### 3.1. Tratamento Semântico de Erros (RFC 9457 — Problem Details)
- É **PROIBIDO** retornar respostas de erro no formato padrão do FastAPI (`{"detail": "..."}`).
- Toda resposta de erro deve utilizar o media type `application/problem+json` e respeitar o modelo `ApiError`:
  - `type`: URI indicando a categoria do problema (ex.: `"/errors/validation-error"`, `"/errors/not-found"`).
  - `title`: Resumo curto e compreensível do status HTTP correspondente.
  - `status`: Código HTTP numérico (ex.: `400`, `404`, `422`).
  - `detail`: Descrição detalhada da ocorrência em português.
  - `instance`: Caminho exato da requisição originária (`request.url.path`).
  - `invalidParams`: Lista de objetos `{"name": "<campo>", "reason": "<motivo>"}` obrigatória em erros de validação (`422`).

### 3.2. Prefixos de Rotas e Endpoints Publicados
- O prefixo oficialmente publicado no contrato OpenAPI é **`/v1/api`** (e **não** `/api/v1`):
  - **Clientes:** `GET /v1/api/clientes`, `POST /v1/api/clientes`, `DELETE /v1/api/clientes/{id_c}`
  - **Produtos:** `GET /v1/api/produtos/`, `POST /v1/api/produtos`, `DELETE /v1/api/produtos/{id_p}`
  - **Usuários:** `GET /v1/api/usuarios`, `POST /v1/api/usuarios`, `DELETE /v1/api/usuarios/{id_u}`
  - **Fornecedores:** `GET /v1/api/fornecedores/{nome_f}`, `POST /v1/api/fornecedores`, `DELETE /v1/api/fornecedores/{id_f}`
- *Nota sobre a rota de produtos:* A listagem possui barra final no contrato (`/v1/api/produtos/`), enquanto as operações POST e DELETE usam `/v1/api/produtos`. O router deve acomodar o caminho publicado e normalizar eventuais redirects sem quebrar chamadas de clientes.

### 3.3. Transição de Status HTTP (Contrato Vigente vs Comportamento-Alvo)
- O OpenAPI exportado declara `200 OK` para todas as operações (inclusive POST e DELETE).
- Para aderência às boas práticas REST recomendadas:
  - Criações (`POST`): Preparadas para retornar `201 Created` e cabeçalho `Location: /v1/api/{recurso}/{id}` após a devida formalização da pendência no Apidog.
  - Exclusões (`DELETE`): Preparadas para retornar `204 No Content` sem corpo de resposta.
  - Operações de leitura e consulta mantêm `200 OK`.

### 3.4. Regras de ORM e Persistência (SQLAlchemy 2.0)
- Usar exclusivamente a sintaxe declarativa do **SQLAlchemy 2.0** (`select()`, `Mapped[...]`, `mapped_column(...)`).
- O vínculo entre produto e fornecedor é estabelecido via chave estrangeira `fornecedor_id` apontando para `fornecedores.id_f`.
- Evitar consultas N+1: ao consultar produtos relacionados a fornecedores, utilizar carregamento explícito via `selectinload` ou `joinedload`.
- Operações de escrita devem conter controle transacional explícito (`db.commit()`, `db.refresh()`, `db.rollback()`).

### 3.5. Blindagem de Dados Sensíveis
- A propriedade `senha_u` do recurso `Usuario` é um dado estritamente de entrada. Ela deve ser hasheada com salt seguro (`bcrypt`) antes da persistência e **nunca** deve ser devolvida em respostas de listagem ou cadastro.

### 3.6. Preservação Fiel dos Nomes de Atributos do Contrato
- Os atributos JSON externos devem respeitar rigorosamente a nomenclatura do contrato:
  - `id_p`, `nome_p`, `descrição_p`, `preço_p`, `quantidadeEstoque`, `categoria`, `status_p`, `fornecedorId`, `data_fabricação`, `data_validade`.
  - `id_f`, `razaoSocial_f`, `nomeFantasia_f`, `cnpj`, `email_f`, `telefone_f`, `endereco_f`, `cidade`, `estado`, `cep`, `status`, `createdAT`, `modifiedAT`.
  - `id_c`, `nome_c`, `email_c`, `telefone_c`, `status_c`.
  - `id_u`, `nome_u`, `email_u`, `senha_u`, `telefone_u`, `status_u`, `createdAt_u`, `updateAt_u`.
- Nos modelos Pydantic v2, utilize aliases (`Field(alias="...")`) para garantir nomes limpos em snake_case internamente no Python enquanto preserva 100% de fidelidade ao contrato JSON externo.
