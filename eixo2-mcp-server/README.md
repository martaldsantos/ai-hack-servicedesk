# Eixo 2 — MCP Server (ServiceDesk como MCP Tools)

## Objetivo

Expor as operações do ServiceDesk como **MCP tools** (Model Context Protocol) via Azure API Management, para que qualquer agente de IA possa descobrir e consumir estas capacidades de forma padronizada.

O resultado esperado é um **MCP Server de ServiceDesk** reutilizável, acessível por qualquer agente — seja Copilot Studio, Microsoft Foundry, ou outro.

---

## Por Onde Começar?

Antes de começar a implementar, sugerimos que a equipa dedique os **primeiros 15-20 minutos** a planear:

1. **Compreender o protocolo MCP** — Se a equipa não está familiarizada, leiam a secção "O que é o MCP?" abaixo e explorem o [MCP Inspector](https://github.com/modelcontextprotocol/inspector) para verem um exemplo real.
2. **Escolher a primeira tool** — Comecem pela `create_ticket`. É a operação mais importante e a que vai ser usada na demo final.
3. **Definir o contrato cedo** — Partilhem com o Eixo 3 o schema da tool e o URL do endpoint assim que possível, mesmo que ainda não esteja funcional. Isto desbloqueia o trabalho deles.
4. **Decidir a abordagem de configuração** — Vão usar o portal do APIM (visual) ou configuração via código/CLI? Ambas funcionam.
5. **Testar incrementalmente** — Não esperem ter as 4 tools para testar. Publiquem a primeira e validem o fluxo completo.

> 💡 **Lembrem-se:** As 4 tools propostas são um ponto de partida. Podem e devem **adicionar novos MCP tools** que façam sentido — por exemplo, `list_areas` (retorna as 9 áreas operacionais), `get_sla_info` (retorna os SLAs por prioridade), ou até tools para **outros sistemas** da organização. Quanto mais tools o MCP Server tiver, mais capaz será qualquer agente que o consuma!

---

## O que é o MCP?

O **Model Context Protocol (MCP)** é um protocolo aberto que padroniza a forma como agentes de IA descobrem e utilizam ferramentas externas. Em vez de cada agente ter integrações custom, o MCP define um contrato universal:

1. O agente liga-se a um **MCP Server**
2. Descobre as **tools** disponíveis (nome, descrição, parâmetros)
3. Invoca a tool pretendida com os parâmetros adequados
4. Recebe a resposta estruturada

**Analogia:** O MCP é para agentes de IA o que o OpenAPI/Swagger é para APIs REST — um contrato que permite interoperabilidade.

---

## Contexto

O agente ServiceDesk executa estas operações:

| Operação | O que faz | Sistema |
|----------|-----------|---------|
| Criar ticket | Cria um work item no Azure DevOps | Azure DevOps |
| Consultar ticket | Obtém detalhes de um ticket existente | Azure DevOps |
| Atualizar ticket | Altera estado, prioridade ou comentários | Azure DevOps |
| Enviar confirmação | Envia email de confirmação ao utilizador | Office 365 |

Cada uma destas operações vai tornar-se uma **MCP tool**.

---

## Tarefas Passo a Passo

### 1. Definir as MCP Tools

Para cada operação, definir o schema da tool com nome, descrição e parâmetros.

**Tools a criar:**

#### a) `create_ticket`
```json
{
  "name": "create_ticket",
  "description": "Cria um novo ticket de suporte técnico no sistema ServiceDesk. Recebe o título, descrição do problema, área operacional e prioridade.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "title": {
        "type": "string",
        "description": "Título resumido do pedido de suporte (ex: 'Outlook não abre')"
      },
      "description": {
        "type": "string",
        "description": "Descrição detalhada do problema reportado pelo utilizador"
      },
      "area": {
        "type": "string",
        "description": "Área operacional do ticket",
        "enum": [
          "ServiceDesk\\Infraestrutura e Redes",
          "ServiceDesk\\Suporte Técnico e Hardware",
          "ServiceDesk\\Gestão de Acessos",
          "ServiceDesk\\Microsoft 365",
          "ServiceDesk\\Aplicações e Software",
          "ServiceDesk\\Aquisições e Licenciamento",
          "ServiceDesk\\Administração de Sistemas",
          "ServiceDesk\\Manutenção e Atualização de Sistemas",
          "ServiceDesk\\Suporte Geral"
        ]
      },
      "priority": {
        "type": "integer",
        "description": "Prioridade do ticket: 1 (crítico, SLA 1h), 2 (alto, SLA 4h), 3 (médio, SLA 1 dia), 4 (baixo, SLA 3 dias)",
        "enum": [1, 2, 3, 4]
      },
      "requester_email": {
        "type": "string",
        "description": "Email do colaborador que fez o pedido"
      }
    },
    "required": ["title", "description", "area", "priority", "requester_email"]
  }
}
```

#### b) `get_ticket_status`
```json
{
  "name": "get_ticket_status",
  "description": "Consulta o estado atual de um ticket de suporte existente. Retorna o estado, prioridade, área e histórico de atualizações.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "ticket_id": {
        "type": "integer",
        "description": "Número identificador do ticket (Work Item ID do Azure DevOps)"
      }
    },
    "required": ["ticket_id"]
  }
}
```

#### c) `update_ticket`
```json
{
  "name": "update_ticket",
  "description": "Atualiza um ticket de suporte existente. Pode alterar o estado, adicionar comentários ou modificar a prioridade.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "ticket_id": {
        "type": "integer",
        "description": "Número identificador do ticket"
      },
      "state": {
        "type": "string",
        "description": "Novo estado do ticket",
        "enum": ["New", "Active", "Resolved", "Closed"]
      },
      "comment": {
        "type": "string",
        "description": "Comentário a adicionar ao ticket"
      },
      "priority": {
        "type": "integer",
        "description": "Nova prioridade (1-4)",
        "enum": [1, 2, 3, 4]
      }
    },
    "required": ["ticket_id"]
  }
}
```

#### d) `send_confirmation_email`
```json
{
  "name": "send_confirmation_email",
  "description": "Envia um email de confirmação ao utilizador com os detalhes do ticket criado, incluindo número, área e SLA estimado.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "to_email": {
        "type": "string",
        "description": "Email do destinatário"
      },
      "ticket_id": {
        "type": "integer",
        "description": "Número do ticket criado"
      },
      "ticket_title": {
        "type": "string",
        "description": "Título do ticket"
      },
      "priority": {
        "type": "integer",
        "description": "Prioridade atribuída (1-4)"
      },
      "sla": {
        "type": "string",
        "description": "SLA estimado (ex: '1 hora', '4 horas', '1 dia', '3 dias')"
      }
    },
    "required": ["to_email", "ticket_id", "ticket_title", "priority", "sla"]
  }
}
```

### 2. Configurar o MCP Server no Azure API Management

O Azure API Management suporta nativamente a exposição de APIs como MCP Servers.

**O que fazer:**

1. **No APIM**, aceder às APIs já importadas pelo Eixo 1 (ou importar diretamente se o Eixo 1 ainda não terminou)
2. **Ativar o MCP Server** no APIM:
   - Ir a **APIs** → selecionar a API do ServiceDesk
   - Configurar o endpoint MCP (`/mcp`)
3. **Mapear cada operação da API para uma MCP tool**, usando as definições acima
4. **Configurar as descrições em português** — isto é fundamental para que os agentes de IA compreendam o contexto

**Endpoint resultante:**
```
https://{apim-name}.azure-api.net/servicedesk/mcp
```

### 3. Configurar Descrições de Qualidade

As descrições das tools são **críticas** — são o que o agente de IA lê para decidir qual tool usar. 

**Boas práticas para descrições:**
- Escrever em **português europeu** (é a língua dos utilizadores)
- Ser **específico** sobre o que a tool faz e quando deve ser usada
- Incluir **exemplos** nos campos de descrição dos parâmetros
- Usar **enums** sempre que possível para restringir valores válidos

**Mau exemplo:**
> "Cria um item"

**Bom exemplo:**
> "Cria um novo ticket de suporte técnico no sistema ServiceDesk. Recebe o título, descrição do problema, área operacional e prioridade. Usar quando um colaborador reporta um problema técnico."

### 4. Testar com um Cliente MCP

**Opção A — Usando o MCP Inspector (recomendado para debug):**
```bash
npx @modelcontextprotocol/inspector
```
Ligar ao endpoint `https://{apim-name}.azure-api.net/servicedesk/mcp` e verificar:
- [ ] As 4 tools são listadas corretamente
- [ ] As descrições aparecem em português
- [ ] Os schemas de input estão corretos
- [ ] Uma chamada a `create_ticket` cria efetivamente um work item no Azure DevOps

**Opção B — Usando curl/HTTP direto:**
```bash
# Listar tools disponíveis
curl -X POST https://{apim-name}.azure-api.net/servicedesk/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "tools/list", "id": 1}'

# Invocar create_ticket
curl -X POST https://{apim-name}.azure-api.net/servicedesk/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "create_ticket",
      "arguments": {
        "title": "Teste MCP - Teams não funciona",
        "description": "O Teams crasha ao abrir reuniões desde a atualização de ontem.",
        "area": "ServiceDesk\\Microsoft 365",
        "priority": 2,
        "requester_email": "utilizador@exemplo.pt"
      }
    },
    "id": 2
  }'
```

### 5. Documentar para o Eixo 3

O Eixo 3 precisa de saber como conectar ao MCP Server. Preparar:

- **URL do MCP Server**: `https://{apim-name}.azure-api.net/servicedesk/mcp`
- **Autenticação**: subscription key ou OAuth token
- **Lista de tools disponíveis** com exemplos de invocação
- **Respostas esperadas** para cada tool

---

## Critério de Sucesso

✅ As 4 MCP tools estão definidas com schemas completos  
✅ O MCP Server está acessível via APIM  
✅ `tools/list` retorna as 4 tools com descrições em português  
✅ `create_ticket` cria efetivamente um work item no Azure DevOps  
🌟 **Bónus:** Todas as 4 tools estão funcionais e testadas  
🌟 **Bónus:** Um agente externo (ex: VS Code com GitHub Copilot) consegue consumir o MCP Server  

---

## Dependências com Outros Eixos

| Dependência | Direção | Detalhe |
|-------------|---------|---------|
| **Eixo 2 depende do Eixo 1** | ← | As APIs governadas pelo APIM são o backend das MCP tools |
| **Eixo 3 depende do Eixo 2** | → | O agente Foundry vai consumir as MCP tools criadas aqui |

**Se o Eixo 1 ainda não estiver pronto:** Podem configurar o MCP Server apontando diretamente ao Azure DevOps e depois redirecionar para o APIM quando o Eixo 1 terminar.

---

## Recursos

- 📖 [MCP — Especificação Oficial](https://modelcontextprotocol.io/)
- 📖 [MCP — Lista de Tools](https://modelcontextprotocol.io/docs/concepts/tools)
- 📖 [Azure API Management — MCP Server](https://learn.microsoft.com/azure/api-management/)
- 📖 [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- 📂 [Azure-Samples/AI-Gateway](https://github.com/Azure-Samples/AI-Gateway)

---

## Dicas

- **Foquem-se no `create_ticket` primeiro** — é a tool essencial para a demo final
- As **descrições em português** são o diferencial — testem se o agente escolhe a tool certa baseado na descrição
- Se tiverem tempo, adicionem uma tool `list_areas` que retorna as 9 áreas operacionais — ajuda o agente a classificar
- Usem o **MCP Inspector** para debug visual antes de passar ao Eixo 3
- Coordenem com o Eixo 1 no **checkpoint das 13:15** para alinhar endpoints
