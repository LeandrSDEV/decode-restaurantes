<img src="assets/capa.png" alt="DECODE — sistema para lanchonetes e restaurantes" width="100%">

# DECODE — sistema para lanchonetes e restaurantes

> **Estudo de caso.** O código do sistema operacional é privado; aqui estão o problema, a arquitetura, as decisões técnicas e as telas.
> O painel administrativo do produto é público: [admin_DecodeBack](https://github.com/LeandrSDEV/admin_DecodeBack) (API) · [admin_DecodeFront](https://github.com/LeandrSDEV/admin_DecodeFront) (front).
> Demonstração e acesso ao código sob solicitação — [LinkedIn](https://www.linkedin.com/in/leandro-js-santos/).

![Java](https://img.shields.io/badge/Java_17-ED8B00?style=flat-square&logo=openjdk&logoColor=white) ![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.3-6DB33F?style=flat-square&logo=springboot&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white) ![Flyway](https://img.shields.io/badge/Flyway-CC0200?style=flat-square&logo=flyway&logoColor=white) ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![MUI](https://img.shields.io/badge/MUI-007FFF?style=flat-square&logo=mui&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

## O problema

Lanchonetes e restaurantes pequenos costumam operar com papel, planilha e conversa de WhatsApp: o pedido da mesa se perde, a taxa de entrega é calculada no olho, a cozinha não vê a fila e o caixa fecha sem relatório. O DECODE junta tudo isso em um único sistema, em tempo real, para eliminar processos manuais e erros operacionais.

## O que o sistema faz

| Atendimento | Operação | Gestão |
| --- | --- | --- |
| Cardápio digital público | PDV (ponto de venda) | Dashboard de faturamento e pedidos |
| Pedido pela mesa via QR Code | Tela da cozinha em tempo real | Estoque, ingredientes, modificadores e variantes |
| Delivery com área de entrega por polígono e taxa | Rotas de entrega | Cupons e promoções |
| Acompanhamento do pedido pelo cliente | Impressão por agente local | Fiscal e TEF |
| Avaliações | Encomendas e pedidos agendados | Usuários e permissões |
| Robô de atendimento no WhatsApp | Pagamento PIX | Mensagens em massa (broadcast) |

## Arquitetura

```mermaid
flowchart LR
    CLI["Cliente<br/>cardápio · mesa · delivery"] --> FE["Front-end<br/>React + TypeScript + MUI"]
    EQ["Equipe<br/>PDV · cozinha · gestão"] --> FE
    FE -- "REST + JWT" --> API["API<br/>Spring Boot 3.3 · Java 17"]
    FE <-- "WebSocket" --> API
    API --> PG[("PostgreSQL<br/>1 schema por estabelecimento")]
    API --> RD[("Redis")]
    API --> EXT["WhatsApp · PIX · TEF · fiscal"]
    API --> PR["Agente de impressão"]
    ADM["Painel admin<br/>admin_DecodeBack"] -- "serviço a serviço" --> API
    API --> OBS["Actuator + Prometheus"]
```

## Decisões técnicas

- **Multi-tenant com schema por estabelecimento.** Um filtro identifica o tenant em cada requisição; o Hibernate troca o `search_path` do PostgreSQL pela conexão (`MultiTenantConnectionProvider`). Os dados de cada cliente ficam isolados numa única instância de banco.
- **Migrações por tenant.** O Flyway versiona o banco (46 migrações) e aplica as migrações em cada schema de estabelecimento.
- **Onboarding automatizado.** O painel admin cria e configura tenants por chamadas serviço a serviço, sem intervenção manual.
- **Tempo real com WebSocket** para a cozinha e o acompanhamento do pedido.
- **Integrações resilientes** com Resilience4j nas chamadas externas.
- **Observabilidade** com Spring Boot Actuator, Micrometer e Prometheus.
- **Segurança** com Spring Security e JWT.
- **Contrato da API** documentado com springdoc OpenAPI.
- **Arquivos:** importação de cardápio por planilha (Apache POI) e geração de PDF (OpenPDF).

## Em números

| 54 | ~376 | 48 | 46 | 34 |
| :---: | :---: | :---: | :---: | :---: |
| controllers REST | classes Java | entidades | migrações Flyway | módulos de tela |

## Telas

<table>
  <tr>
    <td width="50%"><img src="assets/telas/home.png" alt="Página inicial da loja"><br><sub>Página da loja</sub></td>
    <td width="50%"><img src="assets/telas/cardapio-cliente.png" alt="Cardápio digital do cliente"><br><sub>Cardápio digital</sub></td>
  </tr>
  <tr>
    <td><img src="assets/telas/dashboard.png" alt="Dashboard operacional"><br><sub>Dashboard operacional</sub></td>
    <td><img src="assets/telas/pdv.png" alt="Ponto de venda"><br><sub>PDV</sub></td>
  </tr>
  <tr>
    <td><img src="assets/telas/cozinha.png" alt="Tela da cozinha"><br><sub>Cozinha</sub></td>
    <td><img src="assets/telas/delivery.png" alt="Gestão de delivery"><br><sub>Delivery</sub></td>
  </tr>
  <tr>
    <td><img src="assets/telas/areas.png" alt="Áreas de entrega por polígono"><br><sub>Áreas de entrega por polígono</sub></td>
    <td><img src="assets/telas/rotas.png" alt="Rotas de entrega"><br><sub>Rotas de entrega</sub></td>
  </tr>
  <tr>
    <td><img src="assets/telas/estoque.png" alt="Estoque"><br><sub>Estoque</sub></td>
    <td><img src="assets/telas/cupons.png" alt="Cupons e promoções"><br><sub>Cupons e promoções</sub></td>
  </tr>
  <tr>
    <td><img src="assets/telas/configuracao.png" alt="Configuração com QR Code das mesas"><br><sub>Configuração e QR Code das mesas</sub></td>
    <td><img src="assets/telas/whatsapp.png" alt="Robô do WhatsApp"><br><sub>Robô do WhatsApp</sub></td>
  </tr>
</table>

<sub>Telas da loja de demonstração "JS Restaurant". Endereços e dados de clientes aparecem desfocados.</sub>

## Stack

| Camada | Tecnologias |
| --- | --- |
| Back-end | Java 17, Spring Boot 3.3, Spring Security, JWT (jjwt), Spring Data JPA, WebSocket, Resilience4j, springdoc OpenAPI |
| Dados | PostgreSQL (schema por tenant), Flyway, Redis |
| Front-end | React, TypeScript, Material UI, React Router, Recharts, Axios |
| Operação | Docker, Actuator, Micrometer, Prometheus |

---

**Leandro de Jesus Santos** — [LinkedIn](https://www.linkedin.com/in/leandro-js-santos/) · [Portfólio](https://leandrsdev.github.io/portfolio-web/) · [GitHub](https://github.com/LeandrSDEV)
