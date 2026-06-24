# PART I - FIELD PLAYBOOK

## Table of Contents

| # | Section | Description |
|---|---------|-------------|
| 1 | [How to Use This Playbook](#1-how-to-use-this-playbook) | Quick-start guide, layered navigation |
| 2 | [Customer Discovery Framework](#2-customer-discovery-framework) | Discovery questions, signal interpretation |
| 3 | [Core Positioning Message](#3-core-positioning-message) | Key selling points, elevator pitch |
| 4 | [AI Blueprints Overview](#4-ai-patterns-on-oracle-databaseazure----overview) | All 14 blueprints across 3 categories |
|  | [-- Zero Data Movement (Blueprints 1-7)](#zero-data-movement----live-oracle-data) | Live Oracle data, no migration |
|  | [-- Managed Replication (Blueprints 8-10)](#managed-replication----mirrored-analytics-data) | Fabric mirroring + GoldenGate CDC |
|  | [-- AI Enrichment (Blueprints 11-14)](#ai-enrichment-iq----intelligent-data-processing) | Fabric IQ, Foundry IQ, Work IQ, Unified IQ |
| 5 | [Decision Matrix](#5-decision-matrix) | Quick decision guide, detailed comparison, combination blueprints |
| 6 | [Objection Handling](#6-objection-handling) | Common objections and responses |
| 7 | [Field Motion & Engagement Model](#7-field-motion--engagement-model) | Engagement lifecycle, pilot playbook |

**Detailed blueprint docs:**

| Blueprint | Doc |
|---------|-----|
| Blueprint 1: Copilot Studio | [03-copilot-studio.md](03-copilot-studio.md) |
| Blueprints 2-4 and 12: Foundry Agents + Foundry IQ | [04-foundry-agents.md](04-foundry-agents.md) |
| Blueprint 5: Oracle MCP | [05-oracle-mcp.md](05-oracle-mcp.md) |
| Blueprints 8-10 + 8B: Fabric, GoldenGate, Fabric IQ | [06-fabric-data-agents.md](06-fabric-data-agents.md) |
| Blueprint 6: Power Apps | [07-power-apps.md](07-power-apps.md) |
| Blueprint 7: Logic Apps | [08-logic-apps.md](08-logic-apps.md) |
| Blueprints 13-14 + A365: Work IQ, Unified IQ, Control Plane | [09-iq-work-unified.md](09-iq-work-unified.md) |
| Security and Governance | [10-security-governance.md](10-security-governance.md) |
| Reference Architectures | [02-reference-architectures.md](02-reference-architectures.md) |

--

## 1. How to Use This Playbook

This playbook has three layers:

| Layer | Sections | Use When |
|--|--|--|
| **Field** | 1 -- 7 | Customer meetings, executive briefings, partner enablement |
| **Architecture** | 8 -- 16 | Solution architects are in the room; designing credible solutions |

**Quick-start for common scenarios:**

| You need to... | Go to... |
|--|--|
| Position AI on Oracle AI Database@Azure in 5 minutes | Section 3 + Section 4 |
| Help a customer decide which path | Section 5 (Decision Matrix) |
| Design an architecture | [Reference Patterns](02-reference-architectures.md) |
| Build something today | Step-by-Step Guides (coming soon) |
| Implement RAG on Oracle 26ai vectors | [Vector Search (Blueprint 2 -- Foundry Agents)](04-foundry-agents.md#2-ms-foundry--ords-endpoints-rag--vector-search) |
| Wire up MCP tools for agents | [Path 3 -- Oracle MCP](05-oracle-mcp.md) |


## 2. Customer Discovery Framework

> **Rule #1:** Open with discovery, not slides.

### 2.1 Discovery Questions

| # | Question | Why It Matters |
|--|--|--|
| 1 | What problem are you solving with AI? (Q&A, automation, insights, apps) | Maps to the right path |
| 2 | Who is the primary user? (business, ops, developers, DBAs) | Determines Persona and toolchain |
| 3 | Do you require live Oracle data or can you work with analytical copies? | Governs data movement and Fabric scope |
| 4 | What Oracle database version and edition are you running on Oracle AI Database@Azure ? | Oracle 26ai enables native vector search and the common guidance is to use this version for AI capabilities |
| 5 | Is there an existing Microsoft 365 / Power Platform footprint? | Opens Copilot Studio and Power Apps paths |
| 6 | Do you have Microsoft Foundry or Azure OpenAI provisioned? | Qualification for Microsoft Foundry agent path |
| 7 | Are you building for a single use case or a platform play? | Pilot vs platform architecture |
| 8 | What are your data residency and compliance requirements? | Drives network topology and data flow |

### 2.2 Signal Interpretation

| Signal | What It Tells You | Lead With |
|--|--|--|
| "No data movement allowed" | Trust, governance, speed are critical | Copilot Studio gateway / MCP / ORDS with Microsoft Foundry|
| "Business users need answers" | Low-code copilots | Copilot Studio |
| "We want insights and trends" | Analytics + AI | Microsoft Fabric |
| "We're building an app / product" | Pro-dev, APIs, orchestration | Microsoft Foundry |
| "DBAs need automation" | Schema exploration, SQL generation | Oracle MCP Server |
| "We have Oracle 26ai" | Native vector search | Oracle 26ai + Azure OpenAI RAG |
| "We want semantic search" | RAG / vector similarity | Oracle 26ai + Azure OpenAI RAG for vector search |
| "We need multi-step workflows" | Agentic AI with tool orchestration | Microsoft Foundry + MCP |

--

## 3. Core Positioning Message

> **Oracle AI Database@Azure allows customers to build secure, enterprise-grade Agentic AI and RAG solutions on Oracle data using Microsoft AI -- without forcing a single architecture or data movement pattern.**

### Key Selling Points

1. **Start with live Oracle data** -- no ETL prerequisite
2. **Choose your toolchain** -- low-code (Copilot Studio, Power Apps) or pro-code (Microsoft Foundry, MCP, SDKs)
3. **Evolve incrementally** -- from Q&A --> Agents --> Analytics --> AI Applications
4. **Enterprise-grade security** -- Oracle AI Database@Azure network isolation + Entra ID + Oracle DB security
5. **14 proven blueprints across 3 categories** -- Live Data, Mirrored Data, and IQ layers; customers choose based on need
6. **Oracle 26ai native vectors** -- RAG without a separate vector database

### Elevator Pitch (30 seconds)

*"Oracle AI Database@Azure gives your Oracle data a direct line into Microsoft's AI ecosystem. Customers can build copilots, agents, and RAG applications on live/unified Oracle data directly using Copilot Studio, Microsoft Foundry, Oracle MCP tools, Microsoft Fabric, Power Apps, and Oracle 26ai vector search -- all without compromising security"*

--

## 4. AI Blueprints on Oracle AI Database@Azure -- Overview

All blueprints fall into three categories based on how Oracle data flows into the AI ecosystem.

### Zero Data Movement -- Live Oracle Data

Agents query Oracle AI Database@Azure directly at runtime. No data leaves Oracle.

| Blueprint | AI Platform | How It Connects | Surfaces | Value Proposition |
|--|--|--|--|--|
| **1** | **Copilot Studio** | On-Prem Data Gateway / Oracle as Knowledge / Oracle as Tool | Teams, Web, M365 | - Fastest time-to-value (hours)<br/>- No-code builder<br/>- Business users self-serve answers<br/>- Zero data movement |
| **2** | **MS Foundry + MCP** | Oracle MCP Server on Functions / Container Apps (VNET-integrated) | API, M365 Copilot, Agent Store | - NL --> SQL via MCP<br/>- Entra ID SSO/MFA + RBAC<br/>- Private Endpoints end-to-end<br/>- Simplest Foundry blueprint |
| **3** | **MS Foundry + ORDS** | ORDS REST endpoints + Oracle 26ai vector search (RAG); APIM for OAuth2 | API, M365 Copilot, Agent Store | - Governed REST APIs, no raw SQL<br/>- Oracle 26ai RAG / vector search<br/>- APIM enforces Entra ID OAuth2<br/>- All traffic private |
| **4** | **MS Foundry + MCP + ORDS + Foundry IQ** | MCP + ORDS + 26ai vectors + Foundry IQ (Blob, SharePoint, Fabric Files) | API, M365 Copilot, Agent Store | - Complete: structured + unstructured + RAG<br/>- RBAC at every layer<br/>- Separate DB users per tool<br/>- Maximum AI value |
| **5** | **Oracle MCP** (developer) | SQLcl MCP in VS Code or hosted | VS Code, Foundry, Copilot Studio | - Natural language --> SQL in minutes<br/>- Zero infrastructure to start<br/>- Schema discovery on demand<br/>- DBA task automation |
| **6** | **Power Apps** | Oracle Data Gateway / Oracle Connector | Power Platform | - Modernize workflows without rebuilding<br/>- AI Builder for forms & predictions<br/>- Citizen developer friendly<br/>- Incremental AI adoption |
| **7** | **Logic Apps** | Oracle DB Connector / ORDS REST calls | Workflow orchestration, enterprise integration | - Event-driven automation<br/>- 400+ enterprise connectors<br/>- No custom code needed<br/>- Orchestrate Oracle + SaaS + Azure |

### Managed Replication -- Mirrored Analytics Data

Oracle data is replicated into Microsoft Fabric via two paths: Fabric native mirroring (managed private endpoints) or OCI GoldenGate as a Service (real-time CDC). Data Agents on mirrored/replicated data can be published as MCP servers, to Teams, or connected to Copilot Studio and MS Foundry via native connectors.

| Blueprint | AI Platform | How It Connects | Surfaces | Value Proposition |
|--|--|--|--|--|
| **Blueprint 8** | **Mirrored Database + Data Agents** | Oracle --> Fabric Mirroring (private PE) --> Mirrored Database --> Data Agents --> Published as MCP Server / Teams / Copilot Studio / Foundry | Teams, Copilot Studio, Foundry, MCP clients | - NL analytics on mirrored Oracle data<br/>- Data Agent as MCP server for any client<br/>- Publish direct to Teams<br/>- Native connectors to Copilot Studio + Foundry<br/>- Entra ID + private networking end-to-end |
| **Blueprint 8B** | **GoldenGate as a Service + Fabric** | Oracle AI Database@Azure --> OCI GoldenGate (CDC) --> Fabric Lakehouse or Fabric Mirror --> Data Agents | Fabric, Teams, Copilot Studio, Foundry, MCP clients | - Real-time CDC replication (sub-second latency)<br/>- Supports Fabric Lakehouse AND Fabric Mirror as targets<br/>- Data transformations during replication<br/>- 30+ source/target combinations<br/>- Enterprise-grade with conflict detection |
| **Blueprint 9** | **Fabric Mirroring + Data Agents + MS Foundry** | Mirrored Database --> Data Agent --> MS Foundry agent (native connector as tool) | API, M365 Copilot, Agent Store | - Foundry agent uses Data Agent as a tool<br/>- Combine mirrored analytics with live MCP/ORDS<br/>- Best of Fabric + Foundry<br/>- Entra ID RBAC at every layer |
| **Blueprint 10** | **Fabric Mirroring + Data Agents + Copilot Studio** | Mirrored Database --> Data Agent --> Copilot Studio (native connector as tool) | API, M365 Copilot, Agent Store | - Copilot studio Custom Copilot  uses Data Agent as a tool<br/>- Best of Fabric + Copilot Studio<br/>- Entra ID RBAC at every layer |

#### When to Use Fabric Native Mirroring vs GoldenGate as a Service

| Dimension | Blueprint 8: Fabric Native Mirroring | Blueprint 8B: GoldenGate as a Service |
|--|--|--|
| **Setup complexity** | Low -- configure in Fabric UI | Medium -- provision GoldenGate deployment, configure Extract + Replicat |
| **Latency** | Near-real-time (minutes) | Real-time CDC (sub-second to seconds) |
| **Transformation** | No transformation during replication | Yes -- filter, map, merge, transform columns during replication |
| **Target types** | Fabric Mirrored Database only | Fabric Lakehouse OR Fabric Mirror + 30 other targets |
| **Multi-target fan-out** | One target per mirroring config | One source to multiple targets simultaneously |
| **Conflict detection** | N/A (one-way) | Built-in conflict detection and resolution |
| **Schema evolution** | Automatic | Configurable DDL replication |
| **Monitoring** | Fabric workspace monitoring | GoldenGate admin console + OCI monitoring |
| **Cost** | Included in Fabric capacity | OCI GoldenGate service pricing (OCPU-based) |
| **Best for** | Simple analytics mirroring | High-volume CDC, transformations, multi-target, low-latency requirements |

> **References:**
> - [Replicate data from Oracle to Microsoft Fabric Lakehouse](https://docs.oracle.com/en/cloud/paas/goldengate-service/raipm/)

### AI Enrichment: IQ -- Intelligent Data Processing

AI-powered intelligence layers that process, enrich, and surface insights from structured, unstructured, and work data.

| Blueprint | AI Platform | What It Does | Surfaces | Value Proposition |
|--|--|--|--|--|
| **Blueprint 11** | **Fabric IQ** | AI-powered analytics and insights over data in OneLake (mirrored Oracle + other sources) | Fabric, Data Agents | - Automated insight discovery<br/>- AI finds blueprints humans miss<br/>- Multi-source data intelligence<br/>- Scales with Fabric capacity |
| **Blueprint 12** | **Foundry IQ** | Unstructured data processing -- ingests docs from Blob, SharePoint, Fabric Files to ground Foundry agents | Foundry, M365 Copilot | - Unlock PDFs, docs, emails<br/>- Combine unstructured + structured Oracle data<br/>- Single agent, full context<br/>- Enterprise-grade grounding |
| **Blueprint 13** | **Work IQ** | AI-driven productivity insights across M365 work blueprints connected to Oracle business data | M365, Copilot | - Bridge work signals + business data<br/>- Meeting, email, doc intelligence<br/>- Organizational productivity insights<br/>- Connected to Oracle context |
| **Blueprint 14** | **Unified IQ** | All IQ layers combined -- Fabric IQ + Foundry IQ + Work IQ feeding a single intelligent agent | Fabric, Foundry, M365 Copilot | - Complete organizational intelligence<br/>- Structured + unstructured + work signals<br/>- One agent, all context<br/>- Maximum AI value from Oracle investment |

> **See [Reference Architecture Blueprints](02-reference-architectures.md) for detailed Mermaid diagrams of each pattern.**

--

## 5. Decision Matrix

### 5.1 Quick Decision Guide

| Customer Need | Lead With | Why |
|--|--|--|
| Live data Q&A, no movement | **Blueprint 1:** Copilot Studio | Direct gateway, no-code, fastest |
| Custom AI apps -- SQL-first | **Blueprint 2:** MS Foundry + MCP | Natural language --> SQL, schema discovery |
| Custom AI apps -- governed APIs + RAG | **Blueprint 3:** MS Foundry + ORDS | Pre-built REST endpoints, 26ai vector search |
| Full stack agent (structured + unstructured) | **Blueprint 4:** MS Foundry + MCP + ORDS + Foundry IQ | Complete agent across all data types |
| DBA / developer automation | **Blueprint 5:** Oracle MCP | SQL generation, schema exploration |
| Business workflow modernization | **Blueprint 6:** Power Apps | Low-code, incremental AI |
| Enterprise integration / event-driven | **Blueprint 7:** Logic Apps | 400+ connectors, no custom code |
| Cross-source analytics | **Blueprint 8:** Mirrored DB + Data Agents | Managed mirroring, NL queries, publish as MCP/Teams/Copilot Studio/Foundry |
| Real-time CDC to Fabric | **Blueprint 8B:** GoldenGate as a Service + Fabric | Sub-second CDC, transformations, Fabric Lakehouse or Mirror target |
| AI agents on analytics data | **Blueprint 9:** Fabric + Data Agents + Foundry | Data Agent as Foundry tool + live Oracle tools |
| No-code copilots on mirrored data | **Blueprint 10:** Fabric + Data Agents + Copilot Studio | Data Agent as Copilot Studio connector |
| Automated data insights | **Blueprint 11:** Fabric IQ | AI-powered blueprint discovery |
| Unstructured + structured grounding | **Blueprint 12:** Foundry IQ | PDFs, docs, emails + Oracle data |
| Productivity + business data | **Blueprint 13:** Work IQ | M365 signals + Oracle context |
| Full organizational intelligence | **Blueprint 14:** Unified IQ | All IQ layers in one agent |

### 5.2 Detailed Comparison

| Dimension | Blueprint 1 Copilot Studio | Blueprint 2 Foundry+MCP | Blueprint 3 Foundry+ORDS | Blueprint 4 Full Stack | Blueprint 5 Oracle MCP | Blueprint 6 Power Apps | Blueprint 7 Logic Apps | Blueprint 8 Mirrored DB | Blueprint 14 Unified IQ |
|--|--|--|--|--|--|--|--|--|--|
| **Skill level** | Low-code | Pro-dev | Pro-dev | Pro-dev | DBA/Dev | Low-code | Low-code | Data eng | Pro-dev |
| **Data movement** | None | None | None | None | None | None | None | Mirror | Mirror + IQ |
| **Real-time data** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Near-RT | Near-RT |
| **Knowledge grounding** | Oracle as Knowledge | Schema context | Via ORDS + 26ai vectors | Foundry IQ + ORDS + 26ai | Schema context | No | No | Semantic model | All IQ layers |
| **Tool calling** | Connector | MCP | ORDS OpenAPI | MCP + ORDS | Native | No | Connectors | Via Foundry | Via Foundry |
| **Vector search** | No | No | Oracle 26ai | Oracle 26ai | Via SQL | No | No | No | Via Foundry |
| **Unstructured data** | No | No | No | Foundry IQ | No | No | No | No | All IQ |
| **RBAC** | Entra ID + DLP | Entra ID + Azure RBAC | Entra ID + APIM OAuth2 | All layers | DB users | Entra ID | Entra ID | Entra ID | Entra ID |
| **Private networking** | Gateway + PE | VNET + PE | VNET + PE + APIM | VNET + PE + APIM | PE | Gateway + PE | Connectors | Fabric | Fabric + VNET |
| **Cost model** | Per-message | Per-compute | Per-compute + APIM | Per-compute + APIM | Free (local) | Per-user | Per-execution | Fabric CU | Combined |

### 5.3 Combination Blueprints (Most Common)

| Blueprint | Combined | Use Case |
|--|--|--|
| **SQL Agent + Live Data** | Blueprint 2 + Blueprint 5 | Foundry agent using MCP tools for live Oracle queries |
| **RAG Agent** | Blueprint 3 | Foundry agent with ORDS + 26ai vector search for semantic answers |
| **Full Stack Agent** | Blueprint 4 | MCP + ORDS + Foundry IQ -- structured, unstructured, and RAG |
| **Analytics + Agent** | Blueprint 8 + Blueprint 3 | Mirrored data feeding Foundry agents with ORDS for insight delivery |
| **Business + Dev** | Blueprint 1 + Blueprint 2 | Copilot Studio for business; Foundry+MCP for dev |
| **Business Process AI** | Blueprint 6 + Blueprint 1 | Power Apps workflow with Copilot Studio Q&A |
| **Enterprise Automation** | Blueprint 7 + Blueprint 3 | Logic Apps orchestration triggering Foundry agents with ORDS |
| **Complete Intelligence** | Blueprint 4 + Blueprint 8 + Blueprint 13 | Full stack agent + mirrored analytics + all IQ layers |

--

## 6. Objection Handling

| Objection | Response |
|--|--|
| *"We can't move our Oracle data"* | You don't have to. Zero Data Movement blueprints (Blueprints 1-7) all work on live Oracle data with zero data movement. Managed Replication uses managed Fabric mirroring only if analytics require it. |
| *"We're worried about security and governance"* | Oracle AI Database@Azure runs in Azure with full network isolation (Private Endpoints, VNETs). Oracle MCP operates inside Oracle DB security -- it doesn't bypass it. All blueprints support Entra ID. See [Security & Governance](10-security-governance.md) for full guardrails. |
| *"We don't know where to start"* | Start with a single-scenario pilot. Most customers begin with Copilot Studio (Blueprint 1 -- 48-hour proof of value) or an MCP demo (Blueprint 5 -- 2-hour setup in VS Code). |
| *"We already have a vector database"* | Oracle 26ai has native vector support -- one fewer service to manage. But Microsoft Foundry agents can also call external vector DBs via tools. Your choice. |
| *"Is MCP production-ready?"* | MCP is an open standard (Anthropic-initiated, now broadly adopted). For production, host on Azure Container Apps with Entra ID auth and API Management. See [Path 3 -- Oracle MCP](05-oracle-mcp.md). |
| *"What about cost?"* | Start with free/low-cost paths: MCP local is free; Copilot Studio has per-message pricing; Azure OpenAI is pay-per-token. No upfront platform investment required. |
| *"We need multi-agent orchestration"* | Microsoft Foundry supports multi-agent blueprints natively. Combine with MCP tools for Oracle access and 26ai vectors for RAG. See [Combined Patterns](09-combined-patterns.md). |

--

## 7. Field Motion & Engagement Model

### 7.1 Engagement Lifecycle

```
Discover --> Map to Path --> Pilot --> Prove --> Scale
   |              |             |          |         |
   Discovery      Decision     2-4 week   Metrics   Prod
   questions      matrix       POC        & ROI     rollout
```

### 7.2 Pilot Playbook

| Blueprint | Pilot Scope | Duration | Success Metric |
|--|--|--|--|
| Blueprint 1 -- Copilot Studio | 1 business Q&A scenario on live Oracle data | 1 -- 2 days | Users get accurate answers without writing SQL |
| Blueprint 2 -- MS Foundry | 1 agent with 2-3 Oracle tools (MCP + ORDS) | 1 -- 2 weeks | Agent completes a multi-step task end-to-end |
| Blueprint 5 -- Oracle MCP | Developer workspace with MCP in VS Code | 2 hours | Developer generates and runs SQL via natural language |
| Blueprint 6 -- Power Apps | 1 Oracle-connected workflow with AI Builder | 3 -- 5 days | Business process automated with AI assistance |
| Blueprint 7 -- Logic Apps | 1 event-driven Oracle integration flow | 2 -- 3 days | Automated Oracle workflow triggered by events |
| Blueprint 8 -- Mirrored DB | Mirror 1 Oracle schema into Fabric | 1 week | Cross-source dashboard with Data Agent queries |
| Blueprint 14 -- Unified IQ | Connect all IQ layers to a single agent | 2 weeks | Agent answers across structured, unstructured, and work data |
