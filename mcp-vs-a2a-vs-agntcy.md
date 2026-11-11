# MCP vs A2A vs AGNTCY — Comparison

*Status as of November 2026*

## TL;DR

| | **MCP** | **A2A** | **AGNTCY** |
|---|---|---|---|
| **One-liner** | How an agent talks to **tools and data** | How an agent talks to **other agents** | The **infrastructure** that lets agents be found, identified, connected and observed |
| **Direction** | Vertical (agent → tool) | Horizontal (agent ↔ agent) | Cross-cutting layer under/around both |
| **Type** | Protocol | Protocol | Collection of specs + components |

These three are **complementary, not competing**. A typical stack: agents use **MCP** to reach tools, **A2A** to delegate to each other, and **AGNTCY** to discover each other (Directory/OASF), prove identity, carry traffic (SLIM) and observe it all.

---

## Detailed comparison

| Dimension | **MCP servers** (Model Context Protocol) | **A2A** (Agent2Agent Protocol) | **AGNTCY** ("Internet of Agents") |
|---|---|---|---|
| **Originator** | Anthropic (Nov 2024) | Google (Apr 2025) | Cisco / Outshift (early 2025) |
| **Current home / governance** | Linux Foundation — Agentic AI Foundation (AAIF), since Dec 2025 | Linux Foundation (since Jun 2025); reported as moving under AAIF alongside MCP in Aug 2026 | Linux Foundation (since Jul 2025) |
| **Core problem solved** | Standard way to expose tools, data and prompts to an LLM app ("USB-C for AI") | Standard way for independent, opaque agents to discover each other, delegate tasks and exchange results | Shared plumbing for multi-agent systems: discovery, identity, secure messaging, observability |
| **Architecture** | Client ↔ server. Host app (IDE, chat app, agent) runs MCP clients that connect to MCP servers | Client agent ↔ remote agent (peer model) | Set of independently deployable components |
| **Main building blocks** | **Tools** (actions), **Resources** (data), **Prompts** (templates); client features like sampling/elicitation; extensions (e.g. MCP Apps, Tasks) | **Agent Card** (capability manifest), **Task** (with lifecycle states), **Message**, **Parts**, **Artifacts** | **OASF** (agent schema), **Agent Directory Service (ADS)**, **Identity** (verifiable credentials / Agent Badges), **SLIM** (messaging), **Observability SDKs** |
| **Wire format / transport** | JSON-RPC 2.0 over stdio (local) or Streamable HTTP (remote) | JSON-RPC 2.0 over HTTP(S), SSE streaming, gRPC / REST bindings | SLIM (gRPC-based, pub/sub, low-latency, quantum-safe crypto); Directory over gRPC |
| **State model** | **Stateless** since spec 2026-07-28 (initialize handshake and `Mcp-Session-Id` removed; version/capabilities sent per request) | Stateful long-running **tasks** with IDs, status updates, push notifications | Directory is persistent/federated; SLIM supports sessions, groups and pub/sub |
| **Discovery** | Configured by host; registries (e.g. MCP Registry) | Agent Card at well-known URL (`/.well-known/agent-card.json`) | Federated **Agent Directory** of OASF records (can describe A2A agents and MCP servers) |
| **Identity & security** | OAuth 2.1-based authorization; 2026-07-28 adds hardening such as `iss` validation against OAuth mix-up attacks | Enterprise auth schemes declared in Agent Card (OAuth2, OIDC, API keys, mTLS); signed Agent Cards | Verifiable credentials / Agent Badges, SPIFFE/SPIRE for directory federation, MLS-based end-to-end encryption in SLIM |
| **Opacity** | Server exposes its tools explicitly — the model sees and calls them | Agents stay **opaque**: no shared memory, tools or internals | N/A (infrastructure) — wraps whatever protocol agents use |
| **Multi-modality / UX** | Text, images, audio, embedded resources; MCP Apps (interactive UI templates) | Text, files, structured data, forms; streaming updates; human-in-the-loop via task states | Multi-modal and human-in-the-loop messaging via SLIM |
| **Typical use case** | Let Claude/ChatGPT/IDE agent query a DB, read GitHub, call Slack, search docs | A customer-service agent hands a billing task to a billing agent from another vendor | An enterprise catalogs hundreds of agents, verifies who they are, routes encrypted traffic between them and traces it |
| **SDKs / languages** | Official SDKs: TypeScript, Python, Java, Kotlin, C#, Go, Ruby, Rust, Swift, PHP | Official SDKs: Python, JavaScript/TS, Java, Go, .NET | Rust (SLIM), Go (Directory, Identity), Elixir (OASF), Python (observability); Python/JS Directory SDKs |
| **Maturity / adoption** | De-facto standard for tool integration; thousands of servers; supported by Anthropic, OpenAI, Google, Microsoft, AWS and most IDEs | 150+ supporting organizations by Apr 2026; integrated in major cloud platforms | ~150 member orgs; Directory v1.x, OASF v1.0; active but more niche/enterprise |
| **Relationship to the others** | Complements A2A (tools vs agents) | Complements MCP; can be transported over SLIM; Agent Cards can be indexed in AGNTCY Directory | Integrates both: OASF describes A2A agents & MCP servers; provides a Directory MCP server; archived its own Agent Connect Protocol in favor of A2A |
| **License** | Open spec, MIT/Apache SDKs | Apache 2.0 | Apache 2.0 |

---

## When to use which

| If you need to… | Use |
|---|---|
| Give a model/agent access to an API, database, file system or SaaS app | **MCP server** |
| Let your agent delegate work to another team's / vendor's agent | **A2A** |
| Run long-running, multi-step jobs between agents with status tracking | **A2A** |
| Build a searchable catalog of agents and MCP servers across an org | **AGNTCY Directory + OASF** |
| Prove cryptographically which agent is which | **AGNTCY Identity** |
| Secure, low-latency, group/pub-sub agent messaging (incl. quantum-safe) | **AGNTCY SLIM** |
| End-to-end tracing / evaluation of a multi-agent system | **AGNTCY Observability** (OpenTelemetry-based) |

## Stack view

```
┌──────────────────────────────────────────────────────────┐
│                Applications / Agents                     │
├───────────────────────────┬──────────────────────────────┤
│  A2A  (agent ↔ agent)     │  MCP  (agent → tools/data)   │
├───────────────────────────┴──────────────────────────────┤
│  AGNTCY: Directory · OASF · Identity · SLIM · Observability │
├──────────────────────────────────────────────────────────┤
│        Transport: HTTP / gRPC / stdio                    │
└──────────────────────────────────────────────────────────┘
```

---

## Sources

- [MCP 2026-07-28 release — MCP Blog](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- [MCP specification (latest)](https://modelcontextprotocol.io/specification/latest.md)
- [The Register — MCP stateless update](https://www.theregister.com/a/5280027)
- [Linux Foundation — A2A surpasses 150 organizations (Apr 2026)](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year)
- [A2A Project on GitHub](https://github.com/a2aproject)
- [FlowHunt — A2A joins MCP at the Agentic AI Foundation](https://www.flowhunt.io/blog/agentic-ai-foundation-a2a-mcp-standards/)
- [Linux Foundation — Welcomes AGNTCY (Jul 2025)](https://www.linuxfoundation.org/press/linux-foundation-welcomes-the-agntcy-project-to-standardize-open-multi-agent-system-infrastructure-and-break-down-ai-agent-silos)
- [Splunk — AGNTCY: structuring the Internet of Agents](https://www.splunk.com/en_us/blog/artificial-intelligence/agntcy-structure-the-internet-of-agents.html)
- [AGNTCY Directory community](https://dir.agntcy.org/latest/community/)
- [Cisco Outshift — AGNTCY](https://outshift.cisco.com/the-internet-of-agents/agntcy)
