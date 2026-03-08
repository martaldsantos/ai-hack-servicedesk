# A(P)I Hackathon — ServiceDesk AI

**Duração:** 1 dia | **Horário:** 9h–16h

---

## Contexto

A organização está a desenvolver um sistema de ServiceDesk com IA, assente numa arquitetura multi-agente construída em Copilot Studio. O sistema é composto por 9 agentes especializados (4 de orquestração + 5 especialistas por domínio) e permite aos colaboradores submeter pedidos de suporte técnico em linguagem natural, em português europeu.

O hackathon tem como objetivo evoluir esta solução, demonstrando a integração entre **Microsoft Foundry**, **MCP Servers** e **Azure API Management / API Center**.

---

## Agenda

| Hora | Atividade | Duração |
|------|-----------|---------|
| 9:00–9:30 | **Kickoff & Contexto** — Apresentação do sistema ServiceDesk, arquitetura e objetivos do hackathon | 30 min |
| 9:30–10:00 | **Tech Briefing** — Walkthroughs de API Management, MCP Servers e Microsoft Foundry | 30 min |
| 10:00–10:15 | **Formação de Equipas & Atribuição de Eixos** | 15 min |
| 10:15–12:30 | **Hack Sprint 1** — Equipas trabalham no seu eixo | 2h 15min |
| 12:30–13:15 | **Almoço** | 45 min |
| 13:15–13:30 | **Checkpoint** — Standup rápido: bloqueios, progresso, dependências entre equipas | 15 min |
| 13:30–15:00 | **Hack Sprint 2** — Continuar a construir + integração entre eixos | 1h 30min |
| 15:00–15:45 | **Demos** — Cada equipa demonstra o seu trabalho (15 min cada) | 45 min |
| 15:45–16:00 | **Wrap-up & Próximos Passos** — Lições aprendidas, roadmap para produção | 15 min |

---

## 3 Eixos de Trabalho

### Eixo 1 — API Governance (API Center + API Management)

**Objetivo:** Publicar e governar as APIs do Azure DevOps usadas pelo agente ServiceDesk.

**Tarefas:**
1. Registar as 3 operações Azure DevOps (criar/ler/atualizar work items) no **API Center**
2. Importá-las para o **Azure API Management** (Developer tier)
3. Aplicar políticas: rate limiting, autenticação centralizada (Managed Identity ou OAuth), logging de pedidos
4. Testar os endpoints governados — as integrações existentes do agente devem funcionar através do APIM
5. Demonstrar observabilidade: traces e métricas no Application Insights

**Critério de sucesso:** O agente ServiceDesk chama o Azure DevOps através do APIM (não diretamente), com políticas aplicadas.

---

### Eixo 2 — MCP Server (ServiceDesk como MCP Tools)

**Objetivo:** Expor as operações do ServiceDesk como MCP tools via Azure API Management.

**Tarefas:**
1. Definir schemas MCP tool para cada operação: `create_ticket`, `get_ticket_status`, `update_ticket`, `send_confirmation_email`
2. Usar a capacidade MCP Server do APIM para expor estas tools
3. Criar descrições adequadas das tools (em português) para que qualquer agente de IA as possa descobrir e usar
4. Testar com um cliente simples que lista e invoca as MCP tools
5. Documentar o endpoint do MCP Server para consumo pelo Eixo 3

**Critério de sucesso:** Um endpoint MCP Server funcional que qualquer agente de IA pode conectar e descobrir as capacidades do ServiceDesk.

---

### Eixo 3 — Orquestrador em Microsoft Foundry

**Objetivo:** Replicar o orquestrador do Copilot Studio em Microsoft Foundry, consumindo as MCP tools do Eixo 2.

**Tarefas:**
1. Criar um projeto Foundry e configurar o agente orquestrador
2. Conectar ao MCP Server do Eixo 2
3. Implementar o fluxo simplificado de orquestração: receber pedido → classificar → triagem → criar ticket
4. Comparar com o Copilot Studio: flexibilidade, debugging, controlo de fluxo
5. Preparar uma tabela comparativa lado a lado

**Critério de sucesso:** Um agente baseado em Foundry que recebe um pedido de suporte em linguagem natural e cria um ticket via MCP tools.

---

## Matriz de Comparação (Eixo 3)

| Critério | Copilot Studio | Microsoft Foundry |
|----------|---------------|-------------------|
| Complexidade de setup | | |
| Experiência de debug | | |
| Flexibilidade de controlo de fluxo | | |
| Facilidade de integração MCP | | |
| Suporte multi-agente | | |

---

## Ideias para Tornar o Hackathon Mais Dinâmico

1. **Cenário de Demo ao Vivo** — Preparar 3-5 pedidos de suporte realistas em português (ex: "O meu Outlook não abre desde ontem", "Preciso de acesso VPN para trabalhar remotamente") que as equipas devem processar fim-a-fim durante as demos.

2. **Desafio de Integração Cross-Track** — Pontos bónus se os 3 eixos conectarem fim-a-fim: agente Foundry → MCP Server → API governada pelo APIM → ticket criado no Azure DevOps.

3. **Momento "Ticket Real"** — No final, alguém da organização submete um pedido de suporte real pelo novo sistema e assiste à criação do ticket no Azure DevOps em direto.

4. **Starter Kits por Eixo** — Preparar um repositório com:
   - OpenAPI specs para as operações Azure DevOps já usadas
   - Snippets de políticas APIM de exemplo (rate-limiting, auth)
   - Templates de schemas MCP tool
   - Boilerplate de agente Foundry

---

## Checklist Pré-Hackathon

- [ ] Confirmar acesso ao tenant Microsoft Foundry para todos os participantes
- [ ] Provisionar Azure API Management (Developer tier) com suporte MCP ativado
- [ ] Clonar o repositório Azure-Samples/AI-Gateway como material de referência
- [ ] Decidir: usar o Azure DevOps real da organização ou criar um projeto de demo
- [ ] Preparar OpenAPI specs para as 3 operações DevOps + email O365
- [ ] Testar conectividade da rede do local a todos os serviços Azure
- [ ] Criar canal Teams/grupo para comunicação no dia
- [ ] Preparar cenários de teste de pedidos de suporte em português

---

## Pré-requisitos

- Acesso a um tenant de Microsoft Foundry para o dia do hackathon
- Instância de Azure API Management (Developer tier) com suporte a MCP
- Acesso ao repositório Azure-Samples/AI-Gateway como referência de labs
- Confirmação se o Azure DevOps da organização pode ser usado diretamente, ou se se cria um projeto de demo
