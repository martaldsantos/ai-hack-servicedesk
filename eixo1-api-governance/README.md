# Eixo 1 — API Governance (API Center + API Management)

## Objetivo

Publicar e governar as APIs do Azure DevOps utilizadas pelo agente ServiceDesk, usando **Azure API Center** e **Azure API Management (APIM)**.

O resultado esperado é que todas as chamadas do agente ao Azure DevOps passem por uma camada de governação centralizada, com políticas de segurança, rate limiting e observabilidade.

---

## Por Onde Começar?

Antes de mergulhar na implementação, sugerimos que a equipa dedique os **primeiros 15-20 minutos** a planear a abordagem:

1. **Fazer um inventário das APIs** — Listem todas as operações que o agente ServiceDesk já usa (Azure DevOps + O365). Decidam quais são prioritárias para governar primeiro.
2. **Escolher a primeira API** — Recomendamos começar pela operação `criar work item`, pois é a mais usada e a mais visível na demo.
3. **Definir as políticas mínimas** — Que políticas são essenciais para o dia? (sugestão: rate limiting + logging). Podem sempre adicionar mais depois.
4. **Esboçar a arquitetura** — Desenhem num whiteboard o fluxo: Agente → APIM → Azure DevOps. Isto ajuda toda a equipa a ter o mesmo mapa mental.
5. **Dividir tarefas** — Uma pessoa pode trabalhar nas OpenAPI specs enquanto outra configura o APIM.

> 💡 **Lembrem-se:** O que está neste guia é um ponto de partida. Sintam-se à vontade para **adicionar novas APIs** ao catálogo (ex: APIs internas da organização), **criar políticas extra** (ex: caching, transformações), ou **integrar outras fontes** que façam sentido. O hackathon é vosso — sejam criativos!

---

## Contexto

Atualmente, o agente ServiceDesk chama diretamente as APIs do Azure DevOps para:

| Operação | Endpoint Azure DevOps | Descrição |
|----------|----------------------|-----------|
| Criar work item | `POST /{org}/{project}/_apis/wit/workitems/${type}` | Cria um novo ticket de suporte |
| Obter work item | `GET /{org}/{project}/_apis/wit/workitems/{id}` | Consulta detalhes de um ticket existente |
| Atualizar work item | `PATCH /{org}/{project}/_apis/wit/workitems/{id}` | Atualiza campos de um ticket (estado, prioridade, etc.) |

Adicionalmente, utiliza APIs do Office 365 para:

| Operação | Descrição |
|----------|-----------|
| Enviar email | Envia confirmação ao utilizador com número do ticket |
| Obter perfil | Obtém nome e email do utilizador autenticado |

**Problema:** Estas integrações não têm governação centralizada — sem rate limiting, sem logging unificado, sem autenticação centralizada.

---

## Tarefas Passo a Passo

### 1. Preparar as OpenAPI Specs

Antes de registar no API Center, é preciso ter as especificações OpenAPI (Swagger) das operações usadas.

**O que fazer:**
- Criar um ficheiro `servicedesk-devops-api.yaml` com as 3 operações do Azure DevOps
- Incluir schemas de request/response para cada operação
- Documentar os campos obrigatórios para criação de work items (título, descrição, área, prioridade)

**Exemplo de estrutura:**
```yaml
openapi: 3.0.3
info:
  title: ServiceDesk DevOps API
  description: APIs do Azure DevOps usadas pelo agente ServiceDesk
  version: 1.0.0
paths:
  /workitems:
    post:
      operationId: createWorkItem
      summary: Criar um novo ticket de suporte
      # ...
    get:
      operationId: getWorkItem
      summary: Obter detalhes de um ticket
      # ...
    patch:
      operationId: updateWorkItem
      summary: Atualizar um ticket existente
      # ...
```

### 2. Registar as APIs no API Center

O **Azure API Center** é o catálogo centralizado de todas as APIs da organização.

**O que fazer:**
1. Aceder ao Azure API Center no portal Azure
2. Criar um novo ambiente (ex: `servicedesk-dev`)
3. Registar a API `ServiceDesk DevOps API` com:
   - Nome e descrição
   - Upload da OpenAPI spec
   - Metadados: equipa responsável, classificação, ciclo de vida (development)
4. Registar a API `ServiceDesk O365 API` (email + perfil)

**Porquê isto é importante:**
- Qualquer equipa na organização pode descobrir e reutilizar estas APIs
- Garante documentação centralizada e atualizada
- Base para governação no APIM

### 3. Importar para o Azure API Management

O **APIM** é o gateway que vai intermediar todas as chamadas.

**O que fazer:**
1. No Azure API Management (Developer tier), importar as APIs a partir do API Center
2. Configurar o backend para apontar ao Azure DevOps real
3. Criar um **Product** (ex: `servicedesk-apis`) que agrupe estas APIs
4. Gerar uma **subscription key** para o agente ServiceDesk

**Configuração do backend:**
```xml
<set-backend-service base-url="https://dev.azure.com/{org}/{project}/_apis" />
```

### 4. Aplicar Políticas de Governação

Esta é a parte central do Eixo 1 — adicionar valor real à passagem pelo APIM.

**Políticas a implementar:**

#### a) Rate Limiting
```xml
<rate-limit calls="100" renewal-period="60" />
```
Limita a 100 chamadas por minuto — protege contra uso abusivo.

#### b) Autenticação Centralizada
```xml
<authentication-managed-identity resource="499b84ac-1321-427f-aa17-267ca6975798" />
```
Usa Managed Identity para autenticar no Azure DevOps, em vez de PAT tokens espalhados pelo código.

#### c) Logging e Observabilidade
```xml
<trace source="servicedesk-api" severity="information">
    <message>@(context.Request.Body.As<string>(preserveContent: true))</message>
</trace>
```
Envia traces para Application Insights para monitorização em tempo real.

#### d) Validação de Request
```xml
<validate-content unspecified-content-type-action="prevent" 
                  max-size="102400" 
                  size-exceeded-action="prevent" />
```
Garante que os pedidos seguem o schema esperado.

#### e) Transformação (opcional)
```xml
<set-header name="api-version" exists-action="override">
    <value>7.1</value>
</set-header>
```
Garante que a versão da API do Azure DevOps é sempre a correta.

### 5. Testar os Endpoints Governados

**O que fazer:**
1. Usar o **Test Console** do APIM para testar cada operação
2. Verificar que o rate limiting funciona (fazer muitos pedidos seguidos)
3. Confirmar que os logs aparecem no **Application Insights**
4. Testar com um pedido real: criar um work item via APIM e confirmar que aparece no Azure DevOps

**Cenário de teste:**
```bash
# Criar um ticket via APIM
curl -X POST https://{apim-name}.azure-api.net/servicedesk/workitems \
  -H "Ocp-Apim-Subscription-Key: {key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Teste Hackathon - Outlook não abre",
    "description": "O Outlook não inicia desde ontem.",
    "area": "ServiceDesk\\Microsoft 365",
    "priority": 2
  }'
```

### 6. Demonstrar Observabilidade

**O que mostrar na demo:**
- Dashboard no Application Insights com:
  - Número de pedidos por operação
  - Latência média por endpoint
  - Erros (4xx, 5xx) 
  - Rate limiting triggers
- Mapa de dependências mostrando: Agente → APIM → Azure DevOps

---

## Critério de Sucesso

✅ As 3 operações Azure DevOps estão registadas no API Center  
✅ As APIs estão publicadas no APIM com pelo menos 3 políticas ativas  
✅ Um pedido de criação de ticket funciona através do APIM  
✅ Os logs são visíveis no Application Insights  
🌟 **Bónus:** A API de email O365 também está governada no APIM  

---

## Dependências com Outros Eixos

| Dependência | Direção | Detalhe |
|-------------|---------|---------|
| **Eixo 2 precisa do Eixo 1** | → | O MCP Server vai expor as APIs que estão governadas no APIM |
| **Eixo 3 precisa do Eixo 1** | → | O agente Foundry vai fazer chamadas que passam pelo APIM |

**Importante:** Definam os endpoints e subscription keys cedo para que os outros eixos possam avançar em paralelo.

---

## Recursos

- 📖 [Azure API Management — Documentação](https://learn.microsoft.com/azure/api-management/)
- 📖 [Azure API Center — Documentação](https://learn.microsoft.com/azure/api-center/)
- 📖 [Políticas do APIM — Referência](https://learn.microsoft.com/azure/api-management/api-management-policies)
- 📖 [Azure DevOps REST API](https://learn.microsoft.com/rest/api/azure/devops/)
- 📂 [Azure-Samples/AI-Gateway](https://github.com/Azure-Samples/AI-Gateway)

---

## Dicas

- Comecem pelo **criar work item** — é a operação mais importante e a que será usada na demo final
- Usem o **portal do Azure** para configurar políticas visualmente antes de editar o XML
- Se a autenticação Managed Identity for complexa de configurar no tempo disponível, usem um **PAT token** como fallback e adicionem a política de transformação de headers
- Testem cedo e com frequência — não esperem pelo Sprint 2 para validar
