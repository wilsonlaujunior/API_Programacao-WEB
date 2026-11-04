# API de Gestão de Vendas e Entregas

## 1. Visão geral

A **API de Gestão de Vendas e Entregas** é uma API REST desenvolvida
para disponibilizar operações de gerenciamento de clientes, produtos,
usuários e fornecedores.

A API define contratos para troca de informações no formato JSON,
utilizando recursos identificados por URLs e métodos HTTP. Sua
especificação é organizada segundo a abordagem **Design-First**, na qual
os contratos da API são definidos e documentados antes da implementação
dos serviços.

A documentação da API é mantida no **Apidog**, com especificação baseada
em **OpenAPI**.

## 2. Objetivo

O objetivo do projeto é especificar e desenvolver uma API REST para
centralizar operações relacionadas ao cadastro e à consulta de
informações utilizadas em um sistema de gestão de vendas e entregas.

A versão atual do contrato contempla quatro recursos principais:

-   **Clientes:** armazenamento e consulta dos dados dos clientes.
-   **Produtos:** cadastro e consulta de produtos, incluindo informações
    de preço, estoque, categoria, validade e fornecedor.
-   **Usuários:** cadastro e consulta de usuários do sistema.
-   **Fornecedores:** cadastro e consulta de fornecedores e seus dados
    cadastrais.

## 3. Funcionalidades previstas no contrato atual

As operações documentadas para os quatro recursos incluem:

-   Consultar clientes, com possibilidade de filtrar pelo nome.
-   Cadastrar clientes.
-   Excluir clientes pelo identificador.
-   Consultar produtos, com filtros por nome e quantidade em estoque.
-   Cadastrar produtos, associando-os a um fornecedor.
-   Excluir produtos pelo identificador.
-   Consultar usuários, com possibilidade de filtrar pelo nome.
-   Cadastrar usuários.
-   Excluir usuários pelo identificador.
-   Consultar fornecedores pelo nome.
-   Cadastrar fornecedores.
-   Excluir fornecedores pelo identificador.

**Observação:** a especificação atual contempla operações de consulta,
cadastro e exclusão. Operações de atualização, consulta individual por
identificador para todos os recursos, pedidos, vendas e processos de
entrega não fazem parte do contrato documentado nesta versão.

## 4. Arquitetura dos recursos e relacionamentos

A API está organizada em quatro recursos:

### 4.1. Clientes

Representa as pessoas cadastradas como clientes do sistema. Os campos
definidos no contrato incluem:

-   `id_c`: identificador do cliente.
-   `nome_c`: nome do cliente.
-   `email_c`: endereço de e-mail.
-   `telefone_c`: telefone para contato.
-   `status_c`: situação cadastral.

### 4.2. Produtos

Representa os produtos cadastrados para gerenciamento. Os campos
definidos no contrato incluem:

-   `id_p`: identificador do produto.
-   `nome_p`: nome do produto.
-   `descrição_p`: descrição do produto.
-   `preço_p`: preço do produto.
-   `quantidadeEstoque`: quantidade disponível em estoque.
-   `categoria`: categoria do produto.
-   `status_p`: situação do produto.
-   `fornecedorId`: identificador do fornecedor relacionado.
-   `data_fabricação`: data de fabricação.
-   `data_validade`: data de validade.

**Relacionamento:** o campo `fornecedorId` identifica o fornecedor
associado ao produto.

### 4.3. Usuários

Representa os usuários cadastrados para utilização do sistema. Os campos
definidos no contrato incluem:

-   `id_u`: identificador do usuário.
-   `nome_u`: nome do usuário.
-   `email_u`: endereço de e-mail.
-   `senha_u`: senha utilizada no cadastro.
-   `telefone_u`: telefone para contato.
-   `status_u`: situação do usuário.
-   `createdAt_u`: data de criação do registro.
-   `updateAt_u`: data da última atualização.

Por segurança, a senha não deve ser exposta nas respostas da API.

### 4.4. Fornecedores

Representa as empresas ou pessoas responsáveis pelo fornecimento dos
produtos. Os campos definidos no contrato incluem:

-   `id_f`: identificador do fornecedor.
-   `razaoSocial_f`: razão social.
-   `nomeFantasia_f`: nome fantasia.
-   `cnpj`: CNPJ.
-   `email_f`: endereço de e-mail.
-   `telefone_f`: telefone para contato.
-   `endereco_f`: endereço.
-   `cidade`: cidade.
-   `estado`: unidade federativa.
-   `cep`: código de endereçamento postal.
-   `status`: situação cadastral.
-   `createdAT`: data de criação do registro.
-   `modifiedAT`: data da última modificação.

## 5. Endpoints documentados

A versão atual utiliza o prefixo `/v1/api`.

  ---------------------------------------------------------------------------------------
  Recurso           Método            Endpoint                          Operação
  ----------------- ----------------- --------------------------------- -----------------
  Clientes          GET               `/v1/api/clientes`                Listar e filtrar
                                                                        clientes

  Clientes          POST              `/v1/api/clientes`                Cadastrar cliente

  Clientes          DELETE            `/v1/api/clientes/{id_c}`         Excluir cliente

  Produtos          GET               `/v1/api/produtos/`               Listar e filtrar
                                                                        produtos

  Produtos          POST              `/v1/api/produtos`                Cadastrar produto

  Produtos          DELETE            `/v1/api/produtos/{id_p}`         Excluir produto

  Usuários          GET               `/v1/api/usuarios`                Listar e filtrar
                                                                        usuários

  Usuários          POST              `/v1/api/usuarios`                Cadastrar usuário

  Usuários          DELETE            `/v1/api/usuarios/{id_u}`         Excluir usuário

  Fornecedores      GET               `/v1/api/fornecedores/{nome_f}`   Consultar
                                                                        fornecedor pelo
                                                                        nome

  Fornecedores      POST              `/v1/api/fornecedores`            Cadastrar
                                                                        fornecedor

  Fornecedores      DELETE            `/v1/api/fornecedores/{id_f}`     Excluir
                                                                        fornecedor
  ---------------------------------------------------------------------------------------

Os parâmetros de consulta documentados são:

-   Clientes: `nome_c`.
-   Produtos: `nome_p` e `quantidadeEstoque`.
-   Usuários: `nome_u`.

## 6. Padrões técnicos

A especificação e a implementação do projeto seguem os seguintes padrões
e tecnologias:

-   **Estilo arquitetural:** REST.
-   **Formato de dados:** JSON.
-   **Protocolo:** HTTP/HTTPS.
-   **Versionamento das rotas:** `/v1/api`.
-   **Documentação e design do contrato:** Apidog.
-   **Especificação da API:** OpenAPI.
-   **Abordagem de desenvolvimento:** Design-First.
-   **Identificadores:** UUID nos recursos que possuem identificadores
    definidos dessa forma no contrato.

Na implementação prevista pelo projeto, são utilizadas tecnologias como
Python 3.12+, FastAPI, Pydantic v2 e pytest.

## 7. Equipe

  Integrante                    Registro acadêmico
  ----------------------------- --------------------
  Adriana Gasparetto Martelli   2840482423026
  Alexandre Vitoriano Ulloa     2840482423\*\*\*
  Wilson Lau Junior             2840482423013

> O registro acadêmico de Alexandre está parcialmente informado no
> README anterior e deve ser completado antes da entrega final.

## 8. Escopo e evolução

A versão atual da API concentra-se no gerenciamento de clientes,
produtos, usuários e fornecedores, com operações de consulta, cadastro e
exclusão.

A evolução do projeto poderá contemplar funcionalidades relacionadas a
pedidos, vendas, itens de venda, tarefas de entrega, prazos,
acompanhamento de entregas e comprovantes de recebimento. Esses recursos
são possibilidades de evolução e deverão ser especificados no Apidog
antes de serem incorporados ao contrato e à implementação.
