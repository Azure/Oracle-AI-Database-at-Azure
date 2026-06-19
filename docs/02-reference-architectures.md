# Reference Architecture Patterns

Blueprints are organized into three categories based on how Oracle data flows into the Microsoft AI ecosystem.
Platforms used are **Microsoft Foundry, Copilot Studio, Power Apps, Logic apps** for workflows 

--

## Zero Data Movement -- Live Oracle Data

Agents query Oracle data directly running on Oracle AI Database@Azure at runtime. No data leaves Oracle.

| Blueprint | AI Platform | How It Connects | Surfaces | Value Proposition |
|--|--|--|--|--|
| **1** | **Copilot Studio** | Gateway / Oracle as Knowledge / Oracle as Tool | Teams, Web, M365 | - Fastest time-to-value (hours)<br/>- No-code builder<br/>- Business users self-serve answers<br/>- Zero data movement |
| **2** | **MS Foundry** | Agent Framework: Oracle MCP server (hosted on Azure Functions / Azure Container Apps) + ORDS APIs; Knowledge Base (Blob, SharePoint, Fabric Files); Oracle 26ai vectors | API, M365 Copilot, Agent Store | - Full model & tool control<br/>- Multi-agent orchestration<br/>- Production-grade custom AI apps<br/>- Live Oracle data, no migration<br/>- Publish to M365 + Agent Store |
| **3** | **Oracle MCP** (developer) | SQLcl MCP in VS Code or hosted | VS Code, Foundry, Copilot Studio | - Natural language --> SQL in minutes<br/>- Zero infrastructure to start<br/>- Schema discovery on demand<br/>- DBA task automation |
| **4** | **Power Apps** | Gateway / Oracle Connector | Power Platform | - Modernize workflows without rebuilding<br/>- AI Builder for forms & predictions<br/>- Citizen developer friendly<br/>- Incremental AI adoption |
| **5** | **Logic Apps** | Oracle DB Connector (Gateway) / ORDS REST (via HTTP + APIM) | Workflow orchestration, enterprise integration | - Event-driven automation<br/>- 400+ enterprise connectors<br/>- No custom code needed<br/>- Orchestrate Oracle + SaaS + Azure |

--

### Blueprint 1: Copilot Studio + Oracle Connector (On-Prem Data Gateway)

```mermaid
graph TB
 subgraph Users["End Users"]
 BU["Business Users<br/>Teams / Web / M365"]
 end

 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access"]
 end

 subgraph CS["Microsoft Copilot Studio"]
 COP["Custom Copilot"]
 C["Oracle Connector"]
 K["Oracle as Knowledge<br/>Grounds on tables,<br/>views, data"]
 T["Oracle as Tool<br/>Connector actions<br/>called during chat"]
 end

 subgraph VNET["Azure VNET"]
 subgraph GWSub["Gateway Subnet"]
 GATEWAY["On-Premises<br/>Data Gateway<br/>(Azure VM)"]
 end
 subgraph PESub["Private Endpoint Subnet"]
 PE["Private Endpoint"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 DB[("ADBS / Exadata / Base DB<br/>No Public IP")]
 end

 BU -->|SSO| AUTH
 AUTH --> COP
 COP --> C
 C --> K
 C --> T
 K -->|Azure Relay<br/>HTTPS 443| GATEWAY
 T -->|Azure Relay<br/>HTTPS 443| GATEWAY
 GATEWAY -->|Port 1521<br/>Private Endpoint| PE
 PE --> DB
```
Azure Relay is the service that the On-Premises Data Gateway uses to communicate with cloud services like Copilot Studio. This is already how the On-Premises Data Gateway works by default -- you don't configure Azure Relay separately. It's built into the gateway installer.

Here's how it works:

How the Gateway Communicates:
The gateway VM makes an outbound HTTPS connection (port 443) to Azure Relay when it starts up.
This creates a persistent, secure tunnel -- no inbound ports need to be opened on the gateway VM.
When Copilot Studio needs Oracle data, the request flows through this tunnel to the gateway, which then queries Oracle over port 1521 via the Private Endpoint.

--

### Blueprint 1B: Copilot Studio + Oracle MCP (via Azure Functions / Container Apps)

```mermaid
graph TB
    subgraph Users[End Users]
        BU[Business Users<br/>Teams / Web / M365]
    end

    subgraph EntraID[Microsoft Entra ID / OAuth2]
        AUTH[SSO / MFA<br/>Conditional Access]
    end

    subgraph CS[Microsoft Copilot Studio]
        COP[Custom Copilot]
        CC[Copilot Studio MCP Tool]
    end

    subgraph GOV[Governance]
        A365[Agent 365<br/>Approve / Publish / Deploy]
        PURVIEW[Microsoft Purview<br/>Data Map + Catalog]
    end

    subgraph AZ[Azure Integration Layer]
        MCP[Oracle MCP Server<br/>Azure Functions or<br/>Container Apps]
        TOOLS[Oracle Tools<br/>Search / Lookup / Actions]
    end

    subgraph VNET[Azure VNET]
        subgraph PESub[Private Endpoint Subnet]
            PE[Private Endpoint]
        end
    end

    subgraph ODA[Oracle AI Database@Azure]
        DB[(Oracle 26ai DB<br/>No Public IP)]
    end

    BU -->|SSO| AUTH
    AUTH --> COP
    COP --> CC
    CC --> MCP
    MCP --> TOOLS
    TOOLS -->|Port 1521<br/>PE| PE
    PE --> DB
    COP -.->|Publish request| A365
    PURVIEW -.->|Classify| DB
```

> Copilot Studio connects to Oracle via MCP Server hosted on Azure Functions or Container Apps. The MCP tool is added in Copilot Studio --> Tools --> Add new MCP. See [03-copilot-studio.md Blueprint 2](03-copilot-studio.md#pattern-2--copilot-studio--oracle-mcp-through-azure-functionscontainer-apps) for setup steps.

--

### Blueprint 1C: Copilot Studio + Oracle ORDS + Custom Connector

```mermaid
graph TB
    subgraph Users[End Users]
        EU[Business Users / Analysts<br/>Teams / Web / M365]
    end

    subgraph EntraID[Microsoft Entra ID / OAuth2]
        AUTH[SSO / MFA<br/>Conditional Access]
    end

    subgraph CS[Microsoft Copilot Studio]
        COP[Custom Copilot]
        CC1[Custom Connector<br/>ORDS REST APIs<br/>Vector Search + Analytics]
    end

    subgraph GOV[Governance]
        A365[Agent 365<br/>Approve / Publish / Deploy]
        PURVIEW[Microsoft Purview<br/>Data Map + Catalog]
    end

    subgraph VNET[Azure VNET]
        APIM[API Management<br/>OAuth2 + Rate Limiting]
        ORDS[ORDS<br/>VNET-integrated]
        PE[Private Endpoint]
    end

    subgraph AOAI[Azure OpenAI]
        EMB[Embedding API]
    end

    subgraph ODA[Oracle AI Database@Azure]
        ORDS_EP[ORDS REST Endpoints]
        VEC[26ai Vector Engine]
        DATA[(Oracle Data<br/>No Public IP)]
    end

    EU -->|SSO| AUTH
    AUTH --> COP
    COP --> CC1
    CC1 --> APIM
    APIM -->|OAuth2 validated| ORDS
    ORDS -->|Port 1521 PE| PE
    PE --> ORDS_EP
    PE --> VEC
    ORDS_EP --> DATA
    VEC --> DATA
    VEC -.->|Embeddings| EMB
    COP -.->|Publish request| A365
    PURVIEW -.->|Classify| DATA
```

> Copilot Studio connects to Oracle ORDS endpoints via a Custom Connector (imported OpenAPI spec). APIM validates OAuth2 tokens and rate-limits. Supports Oracle 26ai vector search for RAG. See [03-copilot-studio.md Blueprint 3](03-copilot-studio.md#pattern-3--copilot-studio--oracle-ords-api-endpoints--custom-connector) for setup steps.

--

### Blueprint 2: MS Foundry + Oracle MCP Server

```mermaid
graph TB
 subgraph Users["End Users"]
 EU["Developers / Business Users"]
 end

 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access"]
 RBAC_E["RBAC:<br/>Foundry User /<br/>Foundry Contributor"]
 end

 subgraph Foundry["Microsoft Foundry"]
 AGENT["AI Agent"]
 LLM["LLM<br/>GPT-4.1 / o3 / o4-mini"]
 MCP["Oracle MCP Tool"]
 end

 subgraph VNET["Azure VNET"]
 subgraph FuncSub["Functions / Container Apps Subnet"]
 MCPHOST["Hosted MCP Server<br/>Azure Functions or<br/>Container Apps<br/>(VNET-integrated)"]
 end
 subgraph PESub["Private Endpoint Subnet"]
 PE["Private Endpoint"]
 KVPE["Key Vault PE"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 SQL["SQL / PL-SQL Engine"]
 DATA[("Oracle 26ai DB<br/>No Public IP")]
 end

 subgraph Governance["Governance"]
 PV["Microsoft Purview<br/>Data Map + Classification<br/>+ Sensitivity Labels"]
 end

 subgraph Observability["Observability"]
 LA["Log Analytics<br/>Workspace"]
 AM["Azure Monitor<br/>Alerts"]
 end

 subgraph Publish["Published To"]
 M365["M365 Copilot / Teams"]
 STORE["Agent Store"]
 API["API Endpoint"]
 end

 EU -->|SSO| AUTH
 AUTH --> AGENT
 RBAC_E --> AGENT
 AGENT <--> LLM
 AGENT --> MCP
 MCP --> MCPHOST
 MCPHOST -->|Managed Identity| KVPE
 MCPHOST -->|Port 1521<br/>Private Endpoint| PE
 PE --> SQL
 SQL --> DATA
 AGENT --> M365
 AGENT --> STORE
 AGENT --> API
 PV -.->|"Scan & Classify"| DATA
 MCPHOST -.->|"Diagnostics"| LA
 DATA -.->|"Unified Audit"| LA
 LA --> AM
```

--

### Blueprint 3: MS Foundry + Oracle ORDS API Endpoints (RAG / Vector Search)

```mermaid
graph TB
 subgraph Users["End Users"]
 EU["Analysts / App Users"]
 end

 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access"]
 RBAC_E["RBAC:<br/>Foundry User /<br/>ORDS OAuth2 scope"]
 end

 subgraph Foundry["Microsoft Foundry"]
 AGENT["AI Agent"]
 LLM["LLM<br/>GPT-4.1 / o3 / o4-mini"]

 subgraph Tools["Agent Tools (OpenAPI)"]
 OPENAPI["ORDS REST APIs<br/>Pre-built Analytics"]
 VSEARCH["ORDS Vector Search<br/>Oracle 26ai RAG"]
 end
 end

 subgraph VNET["Azure VNET"]
 subgraph APIMSub["APIM Subnet"]
 APIM["API Management<br/>OAuth2 + WAF<br/>Rate limiting"]
 end
 subgraph AISub["AI Subnet"]
 AOAI["Azure OpenAI PE<br/>Embeddings"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 ORDS["ORDS<br/>(runs on Oracle 26ai<br/>instance)"]
 VEC["26ai Vector Engine<br/>VECTOR_DISTANCE"]
 DATA[("Oracle 26ai DB<br/>No Public IP")]
 ORDS -->|localhost| DATA
 end

 subgraph Governance["Governance"]
 PV["Microsoft Purview<br/>Data Map + DLP +<br/>Sensitivity Labels"]
 end

 subgraph Observability["Observability"]
 LA["Log Analytics"]
 AM["Azure Monitor Alerts"]
 end

 subgraph Publish["Published To"]
 M365["M365 Copilot / Teams"]
 STORE["Agent Store"]
 API["API Endpoint"]
 end

 EU -->|SSO| AUTH
 AUTH --> AGENT
 RBAC_E --> AGENT
 AGENT <--> LLM
 AGENT --> OPENAPI
 AGENT --> VSEARCH
 OPENAPI --> APIM
 VSEARCH --> APIM
 APIM -->|"OAuth2 validated<br/>via VNET Peering / PE"| ORDS
 DATA -->|"DBMS_CLOUD<br/>via PE"| AOAI
 VEC --> DATA
 AGENT --> M365
 AGENT --> STORE
 AGENT --> API
 PV -.->|"Scan & Classify<br/>+ DLP Policies"| DATA
 APIM -.->|"Diagnostics"| LA
 DATA -.->|"Unified Audit"| LA
 LA --> AM
```

> **Key difference from previous version**: ORDS runs natively on the Oracle 26ai instance -- no separate Azure compute (App Service / Container Apps) is needed. APIM connects to ORDS via VNET Peering or Private Endpoint. Embedding calls from Oracle to Azure OpenAI also route via Private Endpoint.

--

### Blueprint 4: MS Foundry + Oracle MCP + Oracle ORDS APIs + Foundry IQ (Full Stack)

```mermaid
graph TB
 subgraph Users["End Users"]
 EU["All Personas"]
 end

 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access<br/>+ PIM"]
 RBAC_E["RBAC:<br/>Foundry User/Contributor<br/>ORDS OAuth2 scopes<br/>KB Reader"]
 end

 subgraph Foundry["Microsoft Foundry"]
 AGENT["AI Agent<br/>Orchestrator"]
 LLM["LLM<br/>GPT-4.1 / o3 / o4-mini"]
 ACS["Azure AI<br/>Content Safety"]

 subgraph Tools["Agent Tools"]
 MCP["Oracle MCP<br/>(Functions /<br/>Container Apps)"]
 OPENAPI["ORDS REST APIs"]
 VSEARCH["ORDS Vector Search<br/>Oracle 26ai RAG"]
 end

 subgraph KB["Foundry IQ + Knowledge"]
 FIQ["Foundry IQ<br/>Unstructured Processing"]
 BLOB["Azure Blob"]
 SP["SharePoint"]
 FF["Fabric Files"]
 end
 end

 subgraph VNET["Azure VNET -- Spoke"]
 subgraph FuncSub["MCP Subnet"]
 MCPHOST["Hosted MCP<br/>Functions / Container Apps<br/>VNET-integrated"]
 end
 subgraph APIMSub["APIM Subnet"]
 APIM["API Management<br/>OAuth2 + WAF<br/>Rate Limiting"]
 end
 subgraph PESub["Private Endpoint Subnet"]
 PE["Oracle PE (1521)"]
 KVPE["Key Vault PE"]
 AOAIPE["Azure OpenAI PE"]
 STPE["Storage PE"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 ORDS["ORDS<br/>(runs on Oracle 26ai)"]
 SQL["SQL / PL-SQL"]
 VEC["26ai Vector Engine"]
 VPD["VPD + Data Redaction"]
 DATA[("Oracle 26ai DB<br/>No Public IP")]
 ORDS --> DATA
 end

 subgraph Governance["Governance -- Microsoft Purview"]
 PDM["Data Map +<br/>Classification"]
 PSL["Sensitivity Labels<br/>+ DLP Policies"]
 PLN["Data Lineage"]
 end

 subgraph Observability["Observability"]
 LA["Log Analytics<br/>Workspace"]
 AM["Azure Monitor Alerts"]
 end

 subgraph Publish["Published To"]
 M365["M365 Copilot / Teams"]
 STORE["Agent Store"]
 API["API Endpoint"]
 end

 EU -->|SSO| AUTH
 AUTH --> AGENT
 RBAC_E --> AGENT
 AGENT <--> LLM
 AGENT --> ACS
 AGENT --> MCP
 AGENT --> OPENAPI
 AGENT --> VSEARCH
 AGENT --> FIQ
 FIQ --> BLOB
 FIQ --> SP
 FIQ --> FF
 MCP --> MCPHOST
 OPENAPI --> APIM
 VSEARCH --> APIM
 APIM -->|"OAuth2 validated"| ORDS
 MCPHOST -->|"PE (1521)"| PE
 MCPHOST -->|"Managed Identity"| KVPE
 PE --> SQL
 DATA -->|"DBMS_CLOUD via PE"| AOAIPE
 VEC --> DATA
 SQL --> DATA
 AGENT --> M365
 AGENT --> STORE
 AGENT --> API
 PDM -.->|"Scan"| DATA
 PDM -.->|"Scan"| BLOB
 PDM -.->|"Scan"| SP
 PSL -.->|"Enforce DLP"| AGENT
 PLN -.->|"Track Lineage"| ORDS
 PLN -.->|"Track Lineage"| MCPHOST
 MCPHOST -.->|"Diagnostics"| LA
 APIM -.->|"Diagnostics"| LA
 DATA -.->|"Unified Audit"| LA
 LA --> AM
```

> **Key changes**: ORDS runs natively on the Oracle 26ai instance (no separate Azure compute). Hub-spoke VNET with all Private Endpoints. Microsoft Purview scans Oracle, Blob, and SharePoint -- enforces DLP on agent responses and tracks data lineage from Oracle --> Agent --> User. Log Analytics provides centralized observability across MCP, APIM, and Oracle Unified Audit.

--

### Blueprint 5: Oracle MCP Server (Developer + Hosted)

```mermaid
graph TB
 subgraph Local["Developer Workstation"]
 subgraph VSCode["VS Code"]
 GHC["GitHub Copilot<br/>Agent Mode"]
 SQLDEV["SQL Developer Extension<br/>SQLcl MCP Server<br/>(Auto-registered)"]
 GHC <--> SQLDEV
 end
 end

 subgraph Hosted["Enterprise Hosted"]
 subgraph VNET["Azure VNET"]
 subgraph CompSub["Compute Subnet"]
 FUNC["Azure Functions<br/>MCP Server"]
 CA["Container Apps<br/>MCP Server"]
 end
 subgraph PESub["Private Endpoint Subnet"]
 PE["Oracle PE (1521)"]
 KVPE["Key Vault PE"]
 end
 end
 end

 subgraph Clients["MCP Clients"]
 FOUNDRY["Microsoft Foundry<br/>Agent"]
 CS["Copilot Studio"]
 CUSTOM["Custom Apps"]
 end

 subgraph ODA["Oracle AI Database@Azure"]
 DB[("Oracle 26ai DB<br/>No Public IP")]
 end

 subgraph Observability["Observability"]
 LA["Log Analytics"]
 end

 SQLDEV -->|"Direct connection<br/>(dev only)"| DB
 FUNC -->|"PE (1521)"| PE
 CA -->|"PE (1521)"| PE
 FUNC -->|"Managed Identity"| KVPE
 CA -->|"Managed Identity"| KVPE
 PE --> DB
 FOUNDRY -->|"HTTP Streamable MCP"| FUNC
 FOUNDRY -->|"HTTP Streamable MCP"| CA
 CS -->|"Tool integration"| FUNC
 CUSTOM -->|"HTTP"| CA
 FUNC -.->|"Diagnostics"| LA
 CA -.->|"Diagnostics"| LA
 DB -.->|"Unified Audit"| LA
```

> Oracle MCP Server supports two deployment modes: **local** (VS Code with SQLcl extension -- zero infrastructure, auto-registered for GitHub Copilot Agent Mode) and **hosted** (Azure Functions for serverless/bursty workloads, Azure Container Apps for production/steady traffic). See [Path 3 -- Oracle MCP](05-oracle-mcp.md) for detailed setup.

--

### Blueprint 6: Power Apps + Oracle AI Database@Azure

```mermaid
graph TB
 subgraph Users["End Users"]
 BU["Business Users<br/>Power Platform"]
 end

 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access"]
 end

 subgraph PowerPlatform["Power Platform"]
 PA["Power App<br/>(Canvas / Model-driven)"]
 PAuto["Power Automate<br/>Flows"]
 AIBUILDER["AI Builder<br/>Forms / Predictions"]
 ORA_CONN["Oracle DB Connector"]
 end

 subgraph VNET["Azure VNET"]
 subgraph GWSub["Gateway Subnet"]
 GW["On-Premises<br/>Data Gateway<br/>(VNET-integrated VM)"]
 end
 subgraph PESub["Private Endpoint Subnet"]
 PE["Oracle PE<br/>port 1521"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 DB[("Oracle 26ai DB<br/>No Public IP")]
 end

 subgraph Observability["Observability"]
 LA["Log Analytics"]
 end

 BU -->|SSO| AUTH
 AUTH --> PA
 PA --> ORA_CONN
 PA --> AIBUILDER
 PA --> PAuto
 PAuto --> ORA_CONN
 ORA_CONN -->|"Azure Relay<br/>HTTPS 443"| GW
 GW -->|"Port 1521<br/>Private Endpoint"| PE
 PE --> DB
 DB -.->|"Unified Audit"| LA
```

> Power Apps connects to Oracle AI Database@Azure via the Oracle DB Connector and the On-Premises Data Gateway -- same gateway infrastructure as Copilot Studio (Blueprint 1). AI Builder adds OCR, form processing, and predictions on top of Oracle data. See [Path 5 -- Power Apps](07-power-apps.md) for details.

--

### Blueprint 7: Logic Apps + Oracle AI Database@Azure

#### Option A: Oracle DB Connector (via Gateway)

```mermaid
graph LR
 subgraph Trigger["Trigger"]
 TRIG["Schedule / HTTP /<br/>Event Grid /<br/>Service Bus"]
 end

 subgraph LogicApp["Azure Logic Apps"]
 LA["Logic App<br/>(Standard)"]
 ORA_CONN["Oracle DB<br/>Connector"]
 AI["Azure OpenAI<br/>Action"]
 OUT["Output:<br/>Teams / Email /<br/>Dynamics / SAP"]
 end

 subgraph VNET["Azure VNET"]
 GW["Data Gateway VM<br/>(VNET-integrated)"]
 PE["Oracle PE<br/>port 1521"]
 end

 subgraph ODA["Oracle AI Database@Azure"]
 DB[("Oracle 26ai DB<br/>No Public IP")]
 end

 TRIG --> LA
 LA --> ORA_CONN
 ORA_CONN --> GW
 GW -->|"PE (1521)"| PE
 PE --> DB
 LA --> AI
 LA --> OUT
```

#### Option B: ORDS via HTTP + APIM (Recommended)

```mermaid
graph LR
 subgraph Trigger["Trigger"]
 TRIG["Schedule / HTTP /<br/>Event Grid /<br/>Service Bus"]
 end

 subgraph LogicApp["Azure Logic Apps"]
 LA["Logic App Standard<br/>(VNET-integrated)"]
 HTTP["HTTP Action<br/>+ OAuth2"]
 AI["Azure OpenAI<br/>Action"]
 OUT["Output:<br/>Teams / Email /<br/>Dynamics / SAP"]
 end

 subgraph VNET["Azure VNET"]
 subgraph APIMSub["APIM Subnet"]
 APIM["API Management<br/>OAuth2 + Rate Limit"]
 end
 subgraph PESub["PE Subnet"]
 KVPE["Key Vault PE"]
 AOAIPE["OpenAI PE"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 ORDS["ORDS<br/>(on Oracle 26ai)"]
 DB[("Oracle 26ai DB<br/>+ Vector Search")]
 ORDS -->|localhost| DB
 end

 subgraph Governance["Governance"]
 PV["Microsoft Purview<br/>DLP + Classification"]
 end

 subgraph Observability["Observability"]
 LANA["Log Analytics"]
 end

 TRIG --> LA
 LA --> HTTP
 HTTP -->|"OAuth2 Bearer"| APIM
 APIM -->|"Validated"| ORDS
 LA --> AI
 AI -->|"via PE"| AOAIPE
 LA -->|"Managed Identity"| KVPE
 LA --> OUT
 PV -.->|"Classify"| DB
 LA -.->|"Diagnostics"| LANA
 DB -.->|"Unified Audit"| LANA
```

> **Option B is recommended** -- ORDS runs natively on Oracle 26ai (no gateway infrastructure), APIM enforces OAuth2 + rate limiting, Logic App Standard provides VNET integration for fully private connectivity; supports vector search endpoints. See [Blueprint 7 -- Logic Apps](08-logic-apps.md) for detailed setup, NSG rules, and workflow patterns.

--


## Managed Replication -- Mirrored Analytics Data

Oracle data is replicated into Microsoft Fabric via Mirrored Database for analytics, cross-source joins, and AI grounding. Data Agents built on Mirrored Database can be published as MCP servers, deployed to Teams, or connected to Copilot Studio and MS Foundry via native connectors.

| Blueprint | AI Platform | How It Connects | Surfaces | Value Proposition |
|--|--|--|--|--|
| **Blueprint 8** | **Mirrored Database + Data Agents** | Oracle --> Fabric Mirroring --> Mirrored Database --> Data Agents --> Published as MCP Server / Teams / Copilot Studio / Foundry | Teams, Copilot Studio, Foundry, MCP clients | - Natural language analytics on mirrored Oracle data<br/>- Data Agent as MCP server for any MCP client<br/>- Publish directly to Teams<br/>- Connect to Copilot Studio or Foundry via native connectors<br/>- Cross-source joins<br/>- Entra ID + private networking |
| **Blueprint 8B** | **GoldenGate as a Service + Fabric** | Oracle AI Database@Azure --> OCI GoldenGate (CDC) --> Fabric Lakehouse or Fabric Mirror --> Data Agents / Fabric IQ | Fabric, Teams, Copilot Studio, Foundry, MCP clients | - Real-time CDC replication (sub-second latency)<br/>- Supports Fabric Lakehouse AND Fabric Mirror as targets<br/>- Data transformations during replication<br/>- 30+ source/target combinations<br/>- Enterprise-grade with conflict detection |
| **Blueprint 9** | **Fabric Mirroring + Foundry** | Mirrored Database --> Data Agents --> Foundry agents (via native connector) | API, M365 Copilot, Agent Store | - AI agents grounded in curated analytics<br/>- Data Agent feeds Foundry as a tool<br/>- Best of Fabric + Foundry<br/>- Governed data layer<br/>- Publish insights to M365 Copilot |
| **Blueprint 10: Fabric Mirroring + Data Agents + Copilot Studio** | **Fabric Mirroring + Copilot Studio** | Mirrored Database --> Data Agents --> Copilot Studio custom copilots (via native connector) | Teams, Web, M365 Copilot | - Copilots grounded on mirrored Oracle analytics<br/>- Data Agent feeds Copilot Studio as a connector<br/>- Best of Fabric + Copilot Studio<br/>- No-code copilot builder<br/>- Publish to Teams and M365 Copilot |

--

### Blueprint 8: Mirrored Database + Data Agents

```mermaid
graph TB
 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access"]
 RBAC_E["RBAC:<br/>Fabric Viewer/Contributor<br/>Workspace roles"]
 end

 subgraph VNET["Azure VNET"]
 subgraph PESub["Private Endpoint Subnet"]
 PE_ORA["Oracle<br/>Private Endpoint"]
 end
 subgraph FabricSub["Fabric Managed VNET"]
 FGWY["Fabric Managed<br/>Private Endpoint<br/>to Oracle"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 DB[("ADBS / Exadata / Base DB<br/>No Public IP")]
 end

 subgraph Fabric["Microsoft Fabric"]
 MIRROR["Fabric<br/>Mirroring"]
 MDB["Mirrored<br/>Database"]
 DA["Data Agent<br/>(on Mirrored DB)"]
 end

 subgraph Publish["Data Agent Published As"]
 MCP_PUB["MCP Server<br/>(any MCP client)"]
 TEAMS["Teams<br/>(direct publish)"]
 CS_CONN["Copilot Studio<br/>(native connector)"]
 FOUNDRY_CONN["MS Foundry<br/>(native connector)"]
 end

 subgraph Users["End Users"]
 BA["Business Analyst"]
 DEV["Developer<br/>(VS Code / MCP)"]
 BU["Business User<br/>(Teams)"]
 end

 AUTH --> DA
 RBAC_E --> DA
 DB -->|Private Endpoint| FGWY
 FGWY --> MIRROR
 MIRROR --> MDB
 MDB --> DA
 DA --> MCP_PUB
 DA --> TEAMS
 DA --> CS_CONN
 DA --> FOUNDRY_CONN
 MCP_PUB --> DEV
 TEAMS --> BU
 CS_CONN --> BU
 FOUNDRY_CONN --> BA
```

#### RBAC Model

| Layer | Role | Who Gets It | What It Controls |
|--|--|--|--|
| **Entra ID** | Security Group: `Fabric-DataAgent-Users` | Analysts, business users | Who can query the Data Agent |
| **Entra ID** | Conditional Access | All users | MFA, device compliance |
| **Fabric Workspace** | Viewer | End users | Read-only access to mirrored data + Data Agent |
| **Fabric Workspace** | Contributor | Data engineers | Create/modify mirroring, Data Agents, semantic models |
| **Fabric Workspace** | Admin | Platform admin | Manage workspace security, capacity, private endpoints |
| **Copilot Studio** | Maker / User | Citizen devs / End users | Build copilots using Data Agent connector vs use them |
| **MS Foundry** | Foundry User / Contributor | End users / Developers | Use vs create agents connected to Data Agent |
| **Oracle DB** | Dedicated mirroring user | Fabric mirroring connection | `SELECT` on mirrored schemas only; read-only, no DDL/DML |

#### Private Networking

| # | Control | Details |
|--|--|--|
| 1 | Oracle Private Endpoint | No public IP on Oracle; Fabric connects via managed private endpoint |
| 2 | Fabric Managed VNET | Fabric workspace uses managed private endpoints for outbound connections to Oracle |
| 3 | Mirroring over private path | All data replication flows through private networking -- no public internet |
| 4 | Entra ID auth for Fabric | All Fabric access authenticated via Entra ID SSO/MFA |
| 5 | Workspace-level security | Data Agent inherits Fabric workspace RBAC -- controls who can query |
| 6 | Data Agent publishing security | MCP server / Teams / Copilot Studio access controlled by Entra ID groups |
| 7 | No Oracle credentials in agent | Mirrored Database is the source -- Data Agent never connects to Oracle directly |

#### Data Agent Publishing Options

| Publish As | How | Use Case |
|--|--|--|
| **MCP Server** | Data Agent published as MCP endpoint -- any MCP-compatible client can connect | Developers in VS Code, custom agents, multi-agent workflows |
| **Teams App** | Data Agent published directly into Teams | Business users ask questions in natural language in Teams chat |
| **Copilot Studio Connector** | Native connector in Copilot Studio connects to the Data Agent | Build no-code copilots grounded on mirrored Oracle analytics |
| **MS Foundry Tool** | Native connector in Foundry registers Data Agent as a tool | Foundry agents call Data Agent for analytics alongside other tools |

--

### Blueprint 8B: GoldenGate as a Service + Fabric

```mermaid
graph TB
    subgraph ODA["Oracle AI Database@Azure"]
        DB[("Oracle 26ai DB<br/>Source Tables")]
        REDO["Redo Logs (CDC)"]
    end

    subgraph GG["OCI GoldenGate as a Service"]
        EXT["Extract<br/>Captures CDC"]
        TRAIL["Trail Files"]
        REP["Replicat<br/>Apply + Transform"]
    end

    subgraph FabricTargets["Microsoft Fabric Targets"]
        LH["Fabric Lakehouse<br/>(Delta via ADLS)"]
        FM["Fabric Mirror<br/>(direct target)"]
    end

    subgraph Downstream["Downstream AI"]
        SC["OneLake Shortcut"]
        ONT["Fabric IQ Ontology"]
        DA["Data Agent"]
        FA["Foundry Agent"]
    end

    DB --> REDO
    REDO --> EXT
    EXT --> TRAIL
    TRAIL --> REP
    REP --> LH
    REP --> FM
    LH --> SC
    SC --> ONT
    ONT --> DA
    FM --> DA
    DA --> FA
```

> OCI GoldenGate captures changes from Oracle redo logs (CDC) and delivers them to Fabric Lakehouse (Delta format via ADLS) or Fabric Mirror with sub-second latency. Supports data transformations (filtering, masking, column mapping) during replication. See [06-fabric-data-agents.md Blueprint 8B](06-fabric-data-agents.md#pattern-8b-goldengate-as-a-service----real-time-cdc-to-fabric) for detailed setup, GoldenGate Extract/Replicat configuration, and transformation examples.

--

### Blueprint 9: Fabric Mirroring + Data Agents + Foundry

```mermaid
graph TB
 subgraph EntraID["Microsoft Entra ID"]
 AUTH["SSO / MFA<br/>Conditional Access"]
 RBAC_E["RBAC:<br/>Fabric roles +<br/>Foundry roles"]
 end

 subgraph VNET["Azure VNET"]
 subgraph PESub["Private Endpoint Subnet"]
 PE_ORA["Oracle<br/>Private Endpoint"]
 end
 end

 subgraph ODA["Oracle AI Database@Azure"]
 DB[("ADBS / Exadata / Base DB<br/>No Public IP")]
 end

 subgraph Fabric["Microsoft Fabric"]
 MIRROR["Fabric<br/>Mirroring"]
 MDB["Mirrored<br/>Database"]
 DA["Data Agent<br/>(on Mirrored DB)"]
 end

 subgraph Foundry["Microsoft Foundry"]
 FAGENT["Foundry Agent"]
 LLM["LLM<br/>GPT-4.1 / o3"]
 DA_TOOL["Data Agent<br/>(native connector<br/>as Foundry tool)"]
 OTHER["Other Tools<br/>(MCP / ORDS / Functions)"]
 end

 subgraph Publish["Published To"]
 M365["M365 Copilot / Teams"]
 STORE["Agent Store"]
 API["API Endpoint"]
 end

 EU["End Users"] -->|SSO| AUTH
 AUTH --> FAGENT
 RBAC_E --> FAGENT
 DB -->|Private Endpoint| PE_ORA
 PE_ORA --> MIRROR
 MIRROR --> MDB
 MDB --> DA
 DA --> DA_TOOL
 FAGENT <--> LLM
 FAGENT --> DA_TOOL
 FAGENT --> OTHER
 FAGENT --> M365
 FAGENT --> STORE
 FAGENT --> API
```

#### RBAC Model

| Layer | Role | Who Gets It | What It Controls |
|--|--|--|--|
| **Entra ID** | Security Group: `Foundry-Analytics-Users` | All agent users | Who can use the Foundry agent |
| **Fabric Workspace** | Viewer / Contributor | Data team | Access to mirrored data and Data Agent |
| **MS Foundry** | Foundry User / Contributor | End users / Developers | Use vs create agents |
| **Azure RBAC** | Key Vault Secrets User (if other tools used) | Managed Identity | Read credentials for MCP/ORDS |
| **Oracle DB** | Dedicated mirroring user | Fabric mirroring | `SELECT` on mirrored schemas only |

#### Private Networking

| # | Control | Details |
|--|--|--|
| 1 | Oracle Private Endpoint | No public IP; Fabric mirroring via managed PE |
| 2 | Fabric Managed VNET | Mirroring over private path |
| 3 | Data Agent --> Foundry (native connector) | Internal Azure service-to-service connection -- no public exposure |
| 4 | Other Foundry tools (MCP/ORDS) | VNET-integrated as per Zero Data Movement blueprints |
| 5 | Entra ID everywhere | SSO/MFA for Fabric, Foundry, and published surfaces |

--

### Blueprint 10: Fabric Mirroring + Data Agents + Copilot Studio

```mermaid
graph TB
    subgraph EntraID["Microsoft Entra ID"]
        AUTH["SSO / MFA<br/>Conditional Access"]
    end

    subgraph VNET["Azure VNET"]
        subgraph PESub["Private Endpoint Subnet"]
            PE_ORA["Oracle<br/>Private Endpoint"]
        end
    end

    subgraph ODA["Oracle AI Database@Azure"]
        DB[("ADBS / Exadata / Base DB<br/>No Public IP")]
    end

    subgraph Fabric["Microsoft Fabric"]
        MIRROR["Fabric<br/>Mirroring"]
        MDB["Mirrored<br/>Database"]
        DA["Data Agent<br/>(on Mirrored DB)"]
    end

    subgraph CS["Microsoft Copilot Studio"]
        COP["Custom Copilot"]
        DA_CONN["Data Agent<br/>(native connector<br/>as Knowledge/Tool)"]
    end

    subgraph GOV["Governance"]
        A365["Agent 365<br/>Approve / Publish"]
        PURVIEW["Microsoft Purview"]
    end

    subgraph Publish["Published To"]
        TEAMS["Teams"]
        WEB["Web / Mobile"]
        M365["M365 Copilot"]
    end

    EU["End Users"] -->|SSO| AUTH
    AUTH --> COP
    DB -->|Private Endpoint| PE_ORA
    PE_ORA --> MIRROR
    MIRROR --> MDB
    MDB --> DA
    DA --> DA_CONN
    COP --> DA_CONN
    COP --> TEAMS
    COP --> WEB
    COP --> M365
    COP -.->|Publish request| A365
    PURVIEW -.->|Classify| DB
```

> Copilot Studio connects to Fabric Data Agents via the native connector -- no custom code needed. Business users build no-code copilots grounded on mirrored Oracle analytics data. The Data Agent can be added as a Knowledge source (for grounding) or as a Tool (for action-based queries). Combine with live Oracle data via the Oracle connector (Blueprint 1) for hybrid live + mirrored copilots.

#### RBAC Model

| Layer | Role | Who Gets It | What It Controls |
|--|--|--|--|
| **Entra ID** | Security Group: `CPS-Analytics-Users` | Business users | Who can use the copilot |
| **Fabric Workspace** | Viewer / Contributor | Data team | Access to mirrored data and Data Agent |
| **Copilot Studio** | Maker / User | Citizen devs / End users | Build vs use copilots |
| **Oracle DB** | Dedicated mirroring user | Fabric mirroring | `SELECT` on mirrored schemas only |

#### Private Networking

| # | Control | Details |
|--|--|--|
| 1 | Oracle Private Endpoint | No public IP; Fabric mirroring via managed PE |
| 2 | Fabric Managed VNET | Mirroring over private path |
| 3 | Data Agent --> Copilot Studio (native connector) | Internal Azure service-to-service connection -- no public exposure |
| 4 | Agent365 governance | Agent lifecycle approval before publishing |
| 5 | Entra ID everywhere | SSO/MFA for Fabric, Copilot Studio, and published surfaces |

--

## AI Enrichment: IQ -- Intelligent Data Processing

AI-powered intelligence layers that process, enrich, and surface insights from structured, unstructured, and work data.

| Blueprint | AI Platform | What It Does | Surfaces | Value Proposition |
|--|--|--|--|--|
| **Blueprint 11** | **Fabric IQ** | AI-powered analytics and insights over data in OneLake (mirrored Oracle + other sources) | Fabric, Data Agents | - Automated insight discovery<br/>- AI finds blueprints humans miss<br/>- Multi-source data intelligence<br/>- Scales with Fabric capacity |
| **Blueprint 12** | **Foundry IQ** | Unstructured data processing -- ingests docs from Blob, SharePoint, Fabric Files to ground Foundry agents | Foundry, M365 Copilot | - Unlock PDFs, docs, emails<br/>- Combine unstructured + structured Oracle data<br/>- Single agent, full context<br/>- Enterprise-grade grounding |
| **Blueprint 13** | **Work IQ** | AI-driven productivity insights across M365 work blueprints connected to Oracle business data | M365, Copilot | - Bridge work signals + business data<br/>- Meeting, email, doc intelligence<br/>- Organizational productivity insights<br/>- Connected to Oracle context |
| **Blueprint 14** | **Unified IQ** | All IQ layers combined -- Fabric IQ + Foundry IQ + Work IQ feeding a single intelligent agent | Fabric, Foundry, M365 Copilot | - Complete organizational intelligence<br/>- Structured + unstructured + work signals<br/>- One agent, all context<br/>- Maximum AI value from Oracle investment |

--

### Blueprint 14: Unified IQ -- All Layers Combined

```mermaid
graph TB
 subgraph DataSources["Data Sources"]
 ORA[("Oracle AI Database@Azure<br/>Structured Data")]
 BLOB["Azure Blob<br/>Documents"]
 SP["SharePoint<br/>Files & Sites"]
 M365D["M365<br/>Emails, Meetings, Docs"]
 end

 subgraph IQLayers["IQ Layers"]
 FIQ["Fabric IQ<br/>Analytics Intelligence"]
 FOIQ["Foundry IQ<br/>Unstructured Data Processing"]
 WIQ["Work IQ<br/>Productivity Signals"]
 end

 subgraph Agent["Unified Agent"]
 UA["Foundry Agent<br/>Full Context:<br/>Structured + Unstructured Data + Work"]
 end

 subgraph Publish["Published To"]
 COP["M365 Copilot"]
 TEAMS["Teams"]
 API["API"]
 end

 ORA --> FIQ
 ORA --> FOIQ
 BLOB --> FOIQ
 SP --> FOIQ
 M365D --> WIQ
 FIQ --> UA
 FOIQ --> UA
 WIQ --> UA
 UA --> COP
 UA --> TEAMS
 UA --> API
```
