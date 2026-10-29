# API_Programacao-WEB

SuperEntregas API

1.Visão Geral

A SuperEntregas API é uma interface de programação de aplicações REST destinada ao gerenciamento de serviços de entrega de diferentes categorias de produtos, incluindo alimentos, documentos, compras, presentes e encomendas.

O projeto tem como finalidade centralizar o cadastro de entregadores, o registro de solicitações de entrega, o acompanhamento de status, as informações de origem e destino e os comprovantes de recebimento.

A solução foi idealizada para atender entregadores autônomos, pequenas empresas de logística e aplicações que necessitam integrar serviços de entrega aos seus sistemas.

2.Objetivo

Desenvolver a especificação de uma API REST utilizando a abordagem Design-First, estabelecendo previamente os contratos de comunicação, as estruturas de dados, as regras de validação e os comportamentos esperados das requisições e respostas.

A documentação será desenvolvida no Apidog com utilização de Schemas reutilizáveis e simulação de operações por meio de Mock Server.

3.Funcionalidades

Cadastro, consulta, atualização e exclusão de entregadores.

Cadastro, consulta, atualização e exclusão de entregas.

Associação de um entregador a uma entrega.

Identificação da categoria do produto transportado.

Registro de endereço de coleta e endereço de destino.

Acompanhamento do status da entrega.

Armazenamento de informações de rota e distância estimada.

Registro de comprovante de recebimento.

Consulta de entregas utilizando filtros e paginação.

4.Arquitetura dos Recursos

Entregadores

Representa os profissionais responsáveis pela execução dos serviços de entrega.

Seus principais atributos são identificador, nome, e-mail, telefone, tipo de veículo e status.

Entregas

Representa as solicitações de transporte, incluindo identificação do produto, endereços, destinatário, entregador responsável, status, informações de rota e comprovante.

Relacionamento entre Recursos

Um entregador pode estar associado a várias entregas. Cada entrega pode estar associada a um único entregador ou permanecer sem responsável até a atribuição.

O relacionamento é estabelecido pelo atributo entregadorId presente no recurso Entrega.

5.Padrões Técnicos

Estilo arquitetural: REST

Protocolo: HTTP/HTTPS

Formato de dados: JSON

Versionamento: /api/v1

Identificadores: UUID

Documentação: Apidog

Especificação: OpenAPI

Estratégia de desenvolvimento: Design-First

Simulação: Mock Server do Apidog

6.Equipe

Adriana Gasparetto Martelli 2840482423026
Alexandre Vitoriano Ulloa 2840482423***
Wilson Lau Junior 2840482423013

7.Escopo da Primeira Versão

A primeira versão contempla dois recursos principais, Entregadores e Entregas, com operações CRUD completas.

As funcionalidades de acompanhamento de rota e comprovante de recebimento são inicialmente representadas por atributos do recurso Entrega, permitindo sua evolução futura para serviços específicos.
