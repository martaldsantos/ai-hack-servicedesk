# Eixo 3 — Orquestrador em Microsoft Foundry

## Objetivo

Replicar o agente orquestrador do ServiceDesk em **Microsoft Foundry** (anteriormente Azure AI Foundry), consumindo as **MCP tools** criadas pelo Eixo 2.

O resultado esperado é um agente funcional que recebe pedidos de suporte em linguagem natural, em português, e cria tickets automaticamente — demonstrando uma alternativa ao Copilot Studio com comparação direta.

---

## Por Onde Começar?

Antes de abrir o Foundry, sugerimos que a equipa dedique os **primeiros 15-20 minutos** a planear:

1. **Definir o scope do agente** — Não tentem replicar os 9 agentes do Copilot Studio. Comecem com o fluxo principal: receber pedido → classificar → triagem → criar ticket.
2. **Escrever o system prompt primeiro** — O prompt é o cérebro do agente. Rascunhem-no em conjunto antes de tocar no Foundry. Vejam a sugestão detalhada mais abaixo.
3. **Testar o fluxo conversacional sem MCP** — Configurem o agente com o system prompt e testem se classifica bem e faz as perguntas certas, mesmo sem criar tickets reais. Isto pode avançar enquanto o Eixo 2 prepara o MCP Server.
4. **Coordenar com o Eixo 2** — Peçam-lhes o URL do MCP Server e o schema das tools o mais cedo possível. Se ainda não estiver pronto, usem respostas simuladas.
5. **Preparar a comparação desde o início** — Anotem tempos, dificuldades e impressões à medida que avançam. Será valioso para a tabela comparativa com Copilot Studio.

> 💡 **Lembrem-se:** O agente proposto aqui é uma base. Podem e devem **expandir** — adicionem **novos agentes especializados** (ex: um agente de FAQ que responde a perguntas frequentes sem criar ticket), **novas capacidades** (ex: consultar o estado de tickets existentes), ou experimentem **orquestração multi-agente** no Foundry. Se o Eixo 2 adicionar novos MCP tools, o vosso agente pode consumi-los imediatamente!

---

## O que é o Microsoft Foundry?

O **Microsoft Foundry** é a plataforma unificada da Microsoft para construir aplicações e agentes de IA. Ao contrário do Copilot Studio (low-code, orientado a citizen developers), o Foundry oferece:

- **Controlo total** sobre o fluxo de orquestração
- **Flexibilidade de modelos** — escolher qual LLM usar (GPT-4o, GPT-4.1, etc.)
- **Integração nativa com MCP** — consumir MCP tools como ferramentas do agente
- **Debug avançado** — traces detalhados de cada passo do agente
- **Code-first approach** — Python SDK para máximo controlo

---

## Contexto: Arquitetura Atual (Copilot Studio)

O orquestrador atual no Copilot Studio segue este fluxo:

```
Utilizador → Orquestrador → Classificador → Especialista → Triagem → Validador → Formatador → Azure DevOps + Email
```

Com **9 agentes** especializados:
- **4 de orquestração:** Orquestrador, Classificador, Triagem, Validador, Formatador
- **5 especialistas:** M365, Hardware, Infraestrutura, Acessos, Software

**Para o hackathon**, vamos implementar uma versão **simplificada** em Foundry que cubra o fluxo principal:

```
Utilizador → Agente Foundry → Classificar → Definir Prioridade → Criar Ticket (MCP) → Enviar Email (MCP)
```

---

## Tarefas Passo a Passo

### 1. Configurar o Projeto no Microsoft Foundry

**O que fazer:**

1. Aceder ao [Microsoft Foundry](https://ai.azure.com/)
2. Criar um novo **projeto** (ex: `servicedesk-hackathon`)
3. Selecionar a subscrição Azure e resource group disponibilizados
4. Configurar o **modelo base** — recomendamos **GPT-4o** para melhor compreensão de português
5. Verificar que o projeto tem acesso ao modelo deployado

**Estrutura do projeto:**
```
servicedesk-hackathon/
├── agent.py              # Agente principal (orquestrador)
├── prompts/
│   ├── system.md         # System prompt do agente
│   ├── classifier.md     # Instruções de classificação
│   └── triage.md         # Regras de triagem/prioridade
├── config.py             # Configurações (endpoints, keys)
└── test_scenarios.py     # Cenários de teste
```

### 2. Definir o System Prompt

O system prompt é o que define o comportamento do agente. Deve ser detalhado e em **português europeu**.

**System prompt sugerido:**

```markdown
# Assistente ServiceDesk

Tu és o assistente de suporte técnico da organização. O teu papel é ajudar os 
colaboradores a submeter pedidos de suporte de forma rápida e eficaz.

## O teu fluxo de trabalho:

1. **Receber o pedido** — O utilizador descreve o problema em linguagem natural
2. **Classificar** — Identifica a área operacional correta (ver lista abaixo)
3. **Recolher informação** — Faz perguntas específicas se necessário
4. **Definir prioridade** — Aplica as regras de triagem
5. **Confirmar** — Apresenta o resumo ao utilizador e pede confirmação
6. **Criar ticket** — Usa a tool `create_ticket` para registar no sistema
7. **Notificar** — Usa a tool `send_confirmation_email` para enviar confirmação

## Áreas operacionais:

| Área | Exemplos |
|------|----------|
| ServiceDesk\Infraestrutura e Redes | VPN, servidores, backups, rede |
| ServiceDesk\Suporte Técnico e Hardware | Portáteis, impressoras, periféricos |
| ServiceDesk\Gestão de Acessos | Onboarding, permissões, apps internas |
| ServiceDesk\Microsoft 365 | Outlook, Teams, SharePoint, OneDrive |
| ServiceDesk\Aplicações e Software | Adobe, SPSS, Power BI, apps internas |
| ServiceDesk\Aquisições e Licenciamento | Compras, licenças, renovações |
| ServiceDesk\Administração de Sistemas | Configurações, GPOs, Active Directory |
| ServiceDesk\Manutenção e Atualização de Sistemas | Updates, patches, manutenção |
| ServiceDesk\Suporte Geral | Outros pedidos |

## Regras de prioridade:

| Prioridade | Critério | SLA |
|------------|----------|-----|
| P1 — Crítico | Sistema em baixo, afeta muitos utilizadores | 1 hora |
| P2 — Alto | Utilizador bloqueado, sem workaround | 4 horas |
| P3 — Médio | Problema com workaround disponível | 1 dia útil |
| P4 — Baixo | Pedido de informação, melhoria, ou não urgente | 3 dias úteis |

## Regras importantes:

- Comunica SEMPRE em português europeu
- Se não tiveres confiança suficiente na classificação (< 70%), pergunta ao utilizador
- Apresenta SEMPRE um resumo antes de criar o ticket
- Nunca cries um ticket sem confirmação explícita do utilizador
```

### 3. Conectar as MCP Tools

**O que fazer:**

1. Configurar o agente para usar o **MCP Server** do Eixo 2 como fonte de tools
2. O endpoint será: `https://{apim-name}.azure-api.net/servicedesk/mcp`
3. Configurar a autenticação (subscription key ou OAuth)

**Usando o Azure AI Foundry SDK (Python):**

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

# Inicializar o cliente
project = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint="https://<foundry-project>.api.azureml.ms"
)

# Configurar o agente com MCP tools
agent = project.agents.create_agent(
    model="gpt-4o",
    name="ServiceDesk Assistant",
    instructions=open("prompts/system.md").read(),
    tools=[
        {
            "type": "mcp",
            "mcp": {
                "server_url": "https://{apim-name}.azure-api.net/servicedesk/mcp",
                "headers": {
                    "Ocp-Apim-Subscription-Key": "{subscription-key}"
                }
            }
        }
    ]
)
```

**Alternativa — Configuração no portal Foundry:**
1. Ir ao projeto → **Agents**
2. Criar novo agente
3. Na secção **Tools**, adicionar **MCP Server**
4. Inserir o URL do MCP Server e credenciais
5. Verificar que as tools `create_ticket`, `get_ticket_status`, `update_ticket` e `send_confirmation_email` aparecem

### 4. Implementar o Fluxo de Orquestração

O agente deve seguir este fluxo conversacional:

```
┌─────────────────────────────────────────────┐
│                UTILIZADOR                    │
│  "O meu Outlook não abre desde ontem"       │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│            CLASSIFICAÇÃO                     │
│  Área: ServiceDesk\Microsoft 365             │
│  Confiança: 0.95                             │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│         RECOLHA DE INFORMAÇÃO                │
│  "Que versão do Outlook usa?"                │
│  "Aparece alguma mensagem de erro?"          │
│  "Já tentou reparar o Office?"               │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│              TRIAGEM                         │
│  Prioridade: P2 (bloqueado, sem workaround)  │
│  SLA: 4 horas                                │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│            CONFIRMAÇÃO                       │
│  "Vou criar o seguinte ticket:               │
│   Título: Outlook não abre                   │
│   Área: Microsoft 365                        │
│   Prioridade: P2 (4h)                        │
│   Confirma?"                                 │
└─────────────────┬───────────────────────────┘
                  │ (Sim)
                  ▼
┌─────────────────────────────────────────────┐
│          CREATE_TICKET (MCP)                 │
│  → Work item criado: #12345                  │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│     SEND_CONFIRMATION_EMAIL (MCP)            │
│  → Email enviado com detalhes do ticket      │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│            RESPOSTA FINAL                    │
│  "Ticket #12345 criado com sucesso!          │
│   Receberá um email de confirmação."         │
└─────────────────────────────────────────────┘
```

### 5. Testar com Cenários Reais

Usar estes cenários para validar o agente:

| # | Pedido do utilizador | Área esperada | Prioridade esperada |
|---|---------------------|---------------|---------------------|
| 1 | "O meu Outlook não abre desde ontem, já tentei reiniciar o PC." | Microsoft 365 | P2 |
| 2 | "Preciso de acesso VPN para trabalhar remotamente a partir de amanhã." | Gestão de Acessos | P3 |
| 3 | "A impressora do 3º andar não imprime, aparece erro de papel encravado." | Suporte Técnico e Hardware | P3 |
| 4 | "Preciso do Power BI Desktop instalado no meu portátil." | Aplicações e Software | P4 |
| 5 | "O Teams está a crashar sempre que tento entrar numa reunião." | Microsoft 365 | P2 |
| 6 | "A VPN está em baixo para toda a equipa, ninguém consegue trabalhar." | Infraestrutura e Redes | P1 |

### 6. Preparar a Comparação com Copilot Studio

Preencher a matriz durante o desenvolvimento:

| Critério | Copilot Studio | Microsoft Foundry | Notas |
|----------|---------------|-------------------|-------|
| **Tempo de setup** | | | Quanto tempo até ter um agente funcional |
| **Complexidade de configuração** | | | Nível técnico necessário |
| **Experiência de debug** | | | Facilidade em encontrar e corrigir erros |
| **Flexibilidade de fluxo** | | | Controlo sobre ramificações e loops |
| **Integração MCP** | | | Facilidade em consumir MCP tools |
| **Qualidade das respostas** | | | Em português, com o mesmo prompt |
| **Suporte multi-agente** | | | Facilidade em orquestrar vários agentes |
| **Observabilidade** | | | Logs, traces, métricas disponíveis |
| **Curva de aprendizagem** | | | Para um developer vs. citizen developer |
| **Pronto para produção** | | | Maturidade, escalabilidade, SLA |

**Perguntas-chave para responder na demo:**
1. Para que tipo de casos de uso é melhor cada plataforma?
2. Quando faz sentido migrar de Copilot Studio para Foundry?
3. Podem coexistir? (ex: Copilot Studio para agentes simples, Foundry para orquestração complexa)

---

## Critério de Sucesso

✅ Agente Foundry configurado com system prompt em português  
✅ Conectado ao MCP Server do Eixo 2  
✅ Consegue classificar um pedido na área correta  
✅ Cria um ticket via MCP tool `create_ticket`  
✅ Tabela comparativa com Copilot Studio preenchida  
🌟 **Bónus:** O agente faz perguntas de follow-up antes de criar o ticket   
🌟 **Bónus:** Envia email de confirmação via MCP tool `send_confirmation_email`  
🌟 **Bónus:** Integração fim-a-fim com Eixo 1 e Eixo 2  

---

## Dependências com Outros Eixos

| Dependência | Direção | Detalhe |
|-------------|---------|---------|
| **Eixo 3 depende do Eixo 2** | ← | Precisa do MCP Server funcional para consumir tools |
| **Eixo 3 depende do Eixo 1** | ← | Indiretamente — as APIs governadas são o backend final |

**Se o Eixo 2 ainda não estiver pronto:**
- Comecem pelo **system prompt** e **configuração do agente** — não precisa de MCP
- Testem o fluxo conversacional com respostas simuladas
- Preparem a tabela comparativa enquanto esperam
- Quando o MCP Server estiver pronto, basta adicionar o endpoint

---

## Recursos

- 📖 [Microsoft Foundry — Documentação](https://learn.microsoft.com/azure/ai-foundry/)
- 📖 [Foundry SDK — Python](https://learn.microsoft.com/python/api/overview/azure/ai-projects-readme)
- 📖 [Foundry Agents — Quickstart](https://learn.microsoft.com/azure/ai-foundry/quickstarts/get-started-agents)
- 📖 [MCP — Especificação](https://modelcontextprotocol.io/)
- 📖 [Copilot Studio — Documentação](https://learn.microsoft.com/microsoft-copilot-studio/) (para comparação)

---

## Dicas

- **Invistam tempo no system prompt** — é o que mais impacta a qualidade do agente. Um bom prompt com MCP tools simples é melhor que um prompt fraco com muitas tools.
- **Testem em português desde o início** — não assumam que funciona bem só porque funciona em inglês
- **Comecem sem MCP** — configurem o agente, testem o fluxo conversacional, e só depois adicionem as tools
- **Guardem screenshots/recordings** para a demo — a comparação visual é muito mais impactante do que descrever
- **Não tentem replicar os 9 agentes** — foquem-se no fluxo principal (classificar → triagem → criar ticket)
- Se tiverem tempo extra, experimentem um fluxo **multi-agente no Foundry** com um classificador separado
