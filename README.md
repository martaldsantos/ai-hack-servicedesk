# 🚀 A(P)I Hackathon — ServiceDesk AI

**1-Day Hackathon | 9h–16h**

---

## Bem-vindos!

Este hackathon tem um objetivo claro: **evoluir o Assistente ServiceDesk** — um sistema real e operacional de suporte técnico baseado em IA — utilizando tecnologias Microsoft de última geração.

Ao longo do dia, vamos integrar **Azure API Management**, **MCP Servers** e **Microsoft Foundry** num caso de uso concreto: um agente conversacional que permite submeter pedidos de suporte em linguagem natural e receber um ticket criado automaticamente.

---

## O que é o Assistente ServiceDesk?

É um sistema multi-agente construído em Copilot Studio com **9 agentes especializados** que:

- Recebe pedidos de suporte em **português europeu**
- Classifica automaticamente a área (M365, Hardware, Infraestrutura, Acessos, Software, etc.)
- Define a prioridade e SLA (P1 a P4)
- Cria um ticket no **Azure DevOps** e envia email de confirmação via **Office 365**

**Exemplo:** Um colaborador escreve *"O meu Outlook não abre desde ontem"* → o sistema classifica como Microsoft 365, define prioridade, cria o ticket e envia confirmação — tudo sem formulários.

---

## O Desafio

Vamos trabalhar em **3 eixos paralelos** que, juntos, demonstram uma arquitetura moderna de APIs e agentes de IA:

### Eixo 1 — API Governance 🏛️
> Publicar e governar as APIs do ServiceDesk com **API Center + Azure API Management**

- Registar as operações Azure DevOps no API Center
- Proteger com políticas no APIM (rate limiting, autenticação, observabilidade)
- Resultado: APIs governadas e reutilizáveis

### Eixo 2 — MCP Server 🔌
> Expor o ServiceDesk como **MCP tools** para qualquer agente de IA consumir

- Criar MCP tools: `create_ticket`, `get_ticket_status`, `update_ticket`, `send_confirmation_email`
- Publicar via Azure API Management com descrições em português
- Resultado: um MCP Server de ServiceDesk reutilizável

### Eixo 3 — Microsoft Foundry 🤖
> Replicar o orquestrador em **Microsoft Foundry**, consumindo os MCP tools do Eixo 2

- Construir o agente orquestrador em Foundry
- Comparar com a abordagem atual em Copilot Studio
- Resultado: visão clara de quando usar cada plataforma

---

## Agenda do Dia

| Hora | Atividade |
|------|-----------|
| 9:00–9:30 | Kickoff & Contexto |
| 9:30–10:00 | Tech Briefing (API Management, MCP, Foundry) |
| 10:00–10:15 | Formação de equipas & atribuição de eixos |
| 10:15–12:30 | **Hack Sprint 1** |
| 12:30–13:15 | Almoço |
| 13:15–13:30 | Checkpoint rápido |
| 13:30–15:00 | **Hack Sprint 2** |
| 15:00–15:45 | Demos (15 min por equipa) |
| 15:45–16:00 | Wrap-up & próximos passos |

---

## Equipas

### Estrutura recomendada

Recomendamos **3 equipas**, uma por eixo, com **3-4 pessoas cada**. Cada equipa deve idealmente incluir:

| Papel | Responsabilidade |
|-------|-----------------|
| **Lead Técnico** | Define a abordagem, toma decisões de arquitetura, remove bloqueios |
| **Developer(s)** | Implementa a solução, escreve configurações e código |
| **Integrador / Tester** | Testa a integração com os outros eixos, valida os cenários de teste |

### Composição sugerida

| Equipa | Eixo | Perfil ideal |
|--------|------|-------------|
| **Equipa API** | Eixo 1 — API Governance | Experiência com Azure API Management, políticas de API, OpenAPI specs |
| **Equipa MCP** | Eixo 2 — MCP Server | Familiaridade com APIs REST, definição de schemas, integração de agentes |
| **Equipa Foundry** | Eixo 3 — Microsoft Foundry | Experiência com agentes de IA, orquestração, Copilot Studio ou Foundry |

### Dicas para as equipas

- **Comecem simples.** Foquem-se em ter uma operação funcional (ex: `create_ticket`) antes de adicionar complexidade.
- **Comuniquem entre eixos.** O Eixo 3 depende do Eixo 2, que depende do Eixo 1. Definam contratos cedo (endpoints, schemas).
- **Usem o checkpoint das 13:15** para alinhar dependências e resolver bloqueios.
- **Documentem decisões** à medida que avançam — será útil para as demos e para a produção.

### Pessoas de referência

Se precisarem de ajuda:

| Área | Contacto |
|------|----------|
| Contexto ServiceDesk / Arquitetura atual | Equipa IT da organização |
| Azure API Management / API Center | Equipa Microsoft |
| Microsoft Foundry / MCP | Equipa Microsoft |

---

## Recursos Úteis

- 📂 Repositório de referência: [Azure-Samples/AI-Gateway](https://github.com/Azure-Samples/AI-Gateway)
- 📖 [Azure API Management — Documentação](https://learn.microsoft.com/azure/api-management/)
- 📖 [API Center — Documentação](https://learn.microsoft.com/azure/api-center/)
- 📖 [MCP (Model Context Protocol)](https://modelcontextprotocol.io/)
- 📖 [Microsoft Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

## Critérios de Sucesso

No final do dia, queremos ver:

1. ✅ **Eixo 1:** O agente ServiceDesk a chamar o Azure DevOps através do APIM com políticas ativas
2. ✅ **Eixo 2:** Um MCP Server funcional com pelo menos a operação `create_ticket` disponível
3. ✅ **Eixo 3:** Um agente Foundry a criar um ticket usando MCP tools
4. 🌟 **Bónus:** Os 3 eixos integrados fim-a-fim — pedido em linguagem natural → Foundry → MCP → APIM → ticket no Azure DevOps

---

## Cenários de Teste

Usem estes pedidos de exemplo para testar a solução:

1. *"O meu Outlook não abre desde ontem, já tentei reiniciar o PC."*
2. *"Preciso de acesso VPN para trabalhar remotamente a partir de amanhã."*
3. *"A impressora do 3º andar não imprime, aparece erro de papel encravado."*
4. *"Preciso do Power BI Desktop instalado no meu portátil."*
5. *"O Teams está a crashar sempre que tento entrar numa reunião."*

---

**Bom hack! 💪**
