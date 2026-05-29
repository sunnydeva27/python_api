# MCP Gateway Architecture — Part 1

# Core Architecture + Complete End-to-End Lifecycle

---

# 1. Introduction

The MCP Gateway is a centralized orchestration and execution infrastructure layer designed to enable Large Language Models (LLMs) to dynamically discover, retrieve, route, and execute tools across distributed MCP servers.

The platform is designed for:

* multi-tenant environments
* large-scale tool ecosystems
* distributed execution
* semantic tool retrieval
* scalable orchestration
* future autonomous agent systems

This document explains the COMPLETE request lifecycle from:

* user query
* semantic retrieval
* retriever sub-agents
* LLM reasoning
* orchestration
* routing
* execution
* aggregation
* final response synthesis

This document is intended for:

* platform engineers
* AI infrastructure teams
* backend engineers
* orchestration engineers
* future contributors to the MCP Gateway

---

# 2. System Goals

The gateway is designed to solve the following challenges:

## 2.1 Dynamic Tool Discovery

The system must support:

* hundreds of tools
* dynamic MCP registration
* scalable tool retrieval
* runtime capability discovery

The LLM must NOT statically receive all tools.

---

## 2.2 Distributed MCP Execution

The platform must support:

* multiple MCP servers
* parallel execution
* fault isolation
* distributed orchestration

---

## 2.3 Multi-Tenant Execution

The platform must support:

* multiple workspaces
* multiple projects
* multiple teams
* isolated execution environments

---

## 2.4 Future Agentic Infrastructure

The architecture must evolve toward:

* retriever sub-agents
* planner systems
* workflow DAGs
* autonomous orchestration

without major rewrites.

---

# 3. High-Level Architecture

```text id="flow-001"
User
 ↓
FastAPI API Layer
 ↓
Tenant Context Layer
 ↓
Access Control Layer
 ↓
Semantic Retrieval Layer
 ↓
Retriever Sub-Agent
 ↓
Relevant Tool Selection
 ↓
Main LLM Agent
 ↓
Tool Calls
 ↓
Orchestrator
 ↓
Tenant-Aware Router
 ↓
Executor
 ↓
Distributed MCP Servers
 ↓
Aggregator
 ↓
Tool Results
 ↓
LLM Final Response
 ↓
User
```

---

# 4. Core Architectural Principles

The platform is intentionally layered.

Each layer has a SINGLE responsibility.

| Layer             | Responsibility            |
| ----------------- | ------------------------- |
| Retrieval Layer   | Find relevant tools       |
| Policy Layer      | Enforce permissions       |
| LLM Layer         | Choose tools              |
| Router Layer      | Resolve execution targets |
| Executor Layer    | Execute MCP calls         |
| Aggregation Layer | Merge responses           |

This separation provides:

* scalability
* maintainability
* fault isolation
* independent evolution
* tenant isolation

---

# 5. Multi-Tenant Hierarchy

The system supports hierarchical tenancy.

```text id="tenant-hierarchy"
Workspace
 ↓
Project
 ↓
Chat
```

Each tenant may:

* access different MCP servers
* use different cloud accounts
* enforce different policies
* route tools differently

---

# 6. Example User Query

Example request:

```text id="example-query"
Analyze git commits and check Docker containers.
```

We will use this request throughout the document.

---

# 7. Full End-to-End Lifecycle

The request lifecycle consists of:

1. API ingestion
2. Tenant context generation
3. Access control filtering
4. Semantic retrieval
5. Retriever sub-agent refinement
6. Main LLM reasoning
7. Tool call generation
8. Orchestration
9. Routing
10. Execution
11. Aggregation
12. Final LLM synthesis

---

# 8. API Layer

The API layer is the entry point into the gateway.

Example request:

```json id="api-request"
{
  "workspace_id": "forgeai",
  "project_id": "gateway",
  "team_id": "devops",
  "message": "Analyze git commits and check Docker containers"
}
```

Responsibilities:

* request validation
* authentication
* request normalization
* tenant extraction
* observability initialization

---

# 9. Tenant Context Layer

The tenant context layer generates:

```python id="tenant-context"
tenant_context = {
    "workspace": "forgeai",
    "project": "gateway",
    "team": "devops",
    "chat_id": "chat-001"
}
```

This context propagates through the ENTIRE system.

Tenant context is used for:

* access control
* retrieval filtering
* execution routing
* observability
* audit logging

---

# 10. Message Construction Layer

LangChain constructs:

```python id="messages"
SystemMessage(...)
HumanMessage(...)
```

At this stage:

* no tools have been retrieved
* no execution has happened
* the LLM knows nothing about tools

Only conversational context exists.

---

# 11. Access Control Layer

Before retrieval begins, the system determines:

```text id="access-question"
Which tools can this tenant access?
```

Example:

| Team    | Allowed MCPs |
| ------- | ------------ |
| DevOps  | Docker + AWS |
| Backend | Git          |
| PM Team | Jira         |

This filtering happens BEFORE semantic retrieval.

This guarantees:

* tenant isolation
* least privilege access
* secure execution

---

# 12. Why Semantic Retrieval Exists

Without retrieval:

```text id="bad-flow"
All Tools → LLM
```

Problems:

* context explosion
* token cost increase
* degraded reasoning
* latency increases
* poor tool selection

The system therefore performs:

* semantic retrieval
* embedding search
* tenant-aware filtering

before the LLM call.

---

# 13. Semantic Retrieval Layer

The semantic retrieval layer solves:

```text id="retrieval-question"
Which tools should the LLM SEE?
```

This is one of the MOST critical architectural layers.

---

# 14. Tool Metadata Store

The registry stores:

* tool names
* descriptions
* schemas
* embeddings
* categories
* tags
* MCP mappings

Example:

| Tool               | Description            |
| ------------------ | ---------------------- |
| git_commit_history | Analyze Git commits    |
| docker_ps          | List Docker containers |
| docker_logs        | Fetch Docker logs      |

---

# 15. Embedding Retrieval Lifecycle

User query:

```text id="retrieval-query"
Analyze git commits and check Docker containers.
```

becomes:

* embedding vector
* semantic similarity query

The retrieval layer compares:

* query embeddings
* tool embeddings

using:

* cosine similarity
* pgvector search

---

# 16. Retrieval Flow Diagram

```text id="retrieval-flow"
User Query
 ↓
Embedding Generation
 ↓
Vector Similarity Search
 ↓
Tenant Filtering
 ↓
Candidate Tools
```

---

# 17. Candidate Tool Retrieval

Example candidate tools:

```text id="candidate-tools"
git_commit_history
docker_ps
docker_logs
```

IMPORTANT:

This stage is:

* semantic retrieval
* NOT routing
* NOT execution

---

# 18. Retriever Sub-Agent Layer

The retriever sub-agent is an OPTIONAL advanced reasoning layer.

Purpose:

* refine candidate tools
* improve semantic reasoning
* optimize tool prioritization
* reduce unnecessary tools

---

# 19. Why Retriever Agents Exist

Embeddings alone are insufficient for:

* contextual reasoning
* workflow understanding
* cost optimization
* semantic prioritization

Retriever agents improve:

* retrieval precision
* semantic understanding
* execution efficiency

---

# 20. Retriever Agent Example

Embedding retrieval returns:

```text id="retriever-input"
git_commit_history
docker_ps
docker_logs
```

Retriever agent reasons:

```text id="retriever-reasoning"
docker_logs is probably unnecessary
for the current request.
```

Final retrieved tools:

```text id="retriever-output"
git_commit_history
docker_ps
```

---

# 21. Retriever Agent Flow

```text id="retriever-flow"
Semantic Retrieval
 ↓
Candidate Tools
 ↓
Retriever Agent
 ↓
Refined Tool Set
```

---

# 22. Main LLM Agent

The MAIN LLM now begins.

The LLM receives:

* system prompt
* user message
* relevant tools only

Example:

```text id="llm-context"
AVAILABLE TOOLS:
- git_commit_history
- docker_ps
```

---

# 23. Main LLM Responsibilities

The main LLM is responsible for:

* reasoning
* selecting tools
* generating arguments
* synthesizing responses

The LLM does NOT:

* execute tools
* perform routing
* communicate with MCP servers

---

# 24. Tool Call Generation

The LLM generates structured tool calls.

Example:

```json id="tool-calls"
[
  {
    "tool": "git_commit_history",
    "arguments": {}
  },
  {
    "tool": "docker_ps",
    "arguments": {}
  }
]
```

---

# 25. Tool Call Flow

```text id="tool-call-flow"
Relevant Tools
 ↓
bind_tools()
 ↓
LLM
 ↓
Tool Calls
```

---

# 26. Orchestrator Layer

The orchestrator is the central coordination engine.

Responsibilities:

* validate tool calls
* coordinate execution
* isolate failures
* invoke router
* invoke executor
* invoke aggregator

The orchestrator intentionally does NOT:

* execute HTTP requests
* perform routing internally
* perform retrieval
* aggregate responses directly

---

# 27. Orchestration Flow

```text id="orchestrator-flow"
Tool Calls
 ↓
Validation
 ↓
Routing
 ↓
Execution
 ↓
Aggregation
```

---

# 28. Router Layer

The router solves:

```text id="router-question"
Where should this tool execute?
```

The router does NOT:

* reason semantically
* choose tools
* call LLMs

The router only resolves:

* execution targets
* MCP instances
* environment bindings

---

# 29. Execution Routing Example

Example tool:

```text id="routing-example"
docker_ps
```

Router performs:

```python id="route-example"
route(
    tool_name,
    tenant_context
)
```

Router output:

```json id="route-output"
{
  "server_name": "docker-mcp",
  "url": "http://docker-mcp:8000"
}
```

---

# 30. Router Flow Diagram

```text id="router-flow"
Tool Name
 +
Tenant Context
 ↓
Registry Lookup
 ↓
Execution Target
```

---

# 31. Executor Layer

The executor performs:

* asynchronous execution
* parallel MCP calls
* retries
* timeout handling
* fault isolation

Executor communicates with MCP servers through the MCP client abstraction.

---

# 32. Executor Flow

```text id="executor-flow"
Executor
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tool Execution
```

---

# 33. MCP Server Layer

Examples:

* Git MCP
* Docker MCP
* AWS MCP
* Jira MCP

MCP servers encapsulate:

* infrastructure operations
* integrations
* tool execution
* environment-specific logic

---

# 34. Example MCP Execution

Docker MCP may internally execute:

```bash id="docker-example"
docker ps
```

Git MCP may internally execute:

```bash id="git-example"
git log
```

---

# 35. Aggregator Layer

The aggregator receives:

* successful responses
* failed responses
* partial failures

Responsibilities:

* normalize outputs
* merge responses
* preserve execution state

---

# 36. Aggregator Flow

```text id="aggregator-flow"
Execution Results
 ↓
Normalization
 ↓
Merged Response
```

---

# 37. Tool Results Return To LLM

Execution results return back to the LLM as:

* ToolMessages
* structured execution outputs

Conversation becomes:

```text id="tool-message-flow"
HumanMessage
 ↓
AI Tool Calls
 ↓
Tool Results
```

---

# 38. Final Response Synthesis

The LLM synthesizes:

* execution outputs
* reasoning
* final response

Example:

```text id="final-response"
Git repository contains 240 commits.

Running Docker containers:
- postgres
- redis
```

---

# 39. Final End-to-End Flow

```text id="complete-flow"
User
 ↓
FastAPI
 ↓
Tenant Context
 ↓
Access Control
 ↓
Semantic Retrieval
 ↓
Retriever Sub-Agent
 ↓
Relevant Tools
 ↓
Main LLM
 ↓
Tool Calls
 ↓
Orchestrator
 ↓
Tenant-Aware Router
 ↓
Executor
 ↓
MCP Servers
 ↓
Aggregator
 ↓
LLM Final Response
```

---

# 40. Key Architectural Distinctions

## Retrieval

Question solved:

```text id="retrieval-final"
Which tools should the LLM SEE?
```

---

## Router

Question solved:

```text id="router-final"
Where should selected tools EXECUTE?
```

---

## Planner

Question solved:

```text id="planner-final"
What sequence of actions should happen?
```

These are separate architectural layers.

---

# 41. Current System Status

Already implemented:

* orchestrator
* router
* executor
* aggregator
* registry

Next major milestone:

```text id="next-milestone"
Semantic Tool Retrieval
```

This upgrades the platform from:

* static tool gateway
  to:
* intelligent MCP gateway

---

# 42. Evolution Path

## Phase 1

Execution Gateway

## Phase 2

Semantic Retrieval

## Phase 3

Tenant-Aware Routing

## Phase 4

Retriever Sub-Agents

## Phase 5

Planner Systems

## Phase 6

Autonomous Multi-Agent Infrastructure

---

# 43. Conclusion

The MCP Gateway architecture provides:

* scalable AI infrastructure
* distributed MCP execution
* semantic tool retrieval
* centralized orchestration
* tenant-aware execution
* future autonomous workflow support

The architecture intentionally emphasizes:

* separation of concerns
* scalability
* extensibility
* tenant isolation
* operational resilience
