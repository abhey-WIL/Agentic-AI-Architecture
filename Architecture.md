# Agentic AI Architecture - Orchestrator-Centric Design

------------------------------------------------------------------------

## 1. Introduction

This architecture describes an **orchestrator-driven Agentic AI system**
designed to handle discovery, transactions, and assistance across
**multiple domains and networks** using a **single set of generic
agents**.

The system is built around the principle that:

> **Agents reason. Orchestrators decide. Workflows execute.**

The orchestrator owns control, memory selection, instruction
composition, and agent invocation.\
Agents operate within **strictly bounded contexts** and never
self-direct execution or state access.

------------------------------------------------------------------------

## 2. Agents vs Workflows (Clear Separation of Responsibility)

### 2.1 Agents

Agents are **reasoning units** powered by LLMs.

They are used when: - Interpretation is required - Ambiguity exists -
Planning or inference is needed - Human-like understanding is beneficial

Agents: - Do **not** execute side effects - Do **not** fetch memory - Do
**not** decide what to do next - Operate only on provided inputs

Examples: - Search Agent - Transaction Agent - User Profiling Agent -
Feedback Agent - Payload Creation Agent

------------------------------------------------------------------------

### 2.2 Workflows

Workflows are **deterministic execution paths**.

They are used when: - Steps are known - External systems must be
called - Reliability and observability matter

Workflows: - Are non-LLM - Are retryable - Are auditable - Can be
implemented via systems like **n8n, Temporal, Cadence**

Examples: - Network search execution - Transaction execution - Feedback
ingestion - Async profile updates

------------------------------------------------------------------------

## 3. High-Level Architecture Components

### Core Layers

1.  **Base Orchestrator**
2.  **Agents**
3.  **Guardrails**
4.  **Workflows**
5.  **External Systems & Networks**
6.  **Feedback & Profiling Pipeline**

The **Base Orchestrator** is the brain of the system.

------------------------------------------------------------------------

## 4. Base Orchestrator - Workflow

The Base Orchestrator is **not an agent**.\
It **may use lightweight LLM calls**, but it does **not perform domain
reasoning**.

Its role is to: - Interpret intent - Build execution context - Select
memory - Compose instructions - Choose agents - Decide execution paths

[Mermaid link](https://mermaid.live/edit#pako:eNptU01vozAQ_SuWzyRNUgIURZEg0a5yYCule1rSgwMTQAt21h_ZpG3-eweTVFSCi_F45s289-x3mokcaEgPtfiflUxq8jvecYKfMvtCsmNJ4uc0ZgrIs8xKUFoyLeRrl9N-2026hX8GT8iGFxKUWuzlw3KhGlbXy19C4lq9Aan40WgSac2ykjSgWc40Wzx0aT28zTbdcA1ck60wGmQfbVUzparDBcFsxgvUkGnSDj-AtIrTlcDEsyaxqer8O5YNEThDZnQlOMm61AGcJEkTaIS8kIRxVnyHuY0gcTkxHKmxmUO8EuSF8pnMthuCuigNDZGmBkWioiUoRQ3kB9IjeyjZqRKSIKXWBFRADXSJ1mlXua7UkWl0bGha1qb0qvtuktFoiSb0DbGhVdxX1oaSpC9SV5j0GdtQtO5CwPN7p2jdnnzYSTf8JDLWahKirV8CKefuiHPT9INEP2_kRug8w-vYqfRKHVrIKqchFoNDG8A7127pe9tuR3UJDexoiL85k393dMevWHNk_I8Qzb1MClOUNDywWuHOHPF-wrpi-Aiar6hEEiBXwnBNw2ngWRAavtMzDWe-O_Yn_tSdeYHrzieuQy8YDcZTfz59CjzfnU8D359fHfpm207GQRB4TzNv4j36vuf7jw5lRouXC8_uQ0Fe4XNLukdq3-r1E-P5Lig)

------------------------------------------------------------------------

### 4.1 Base Orchestrator Components

#### 1. Request Ingress

-   Accepts raw user input
-   Normalizes metadata (user, session, channel)

**Output**

``` json
{
  "rawQuery": "...",
  "userId": "...",
  "sessionId": "...",
  "channel": "web"
}
```

------------------------------------------------------------------------

#### 2. Intent Router

Determines **what kind of interaction** this is.

-   SEARCH
-   TRANSACTION
-   HELP
-   FEEDBACK

Uses: - Rules - Lightweight classification models / LLM (bounded)

**Output**

``` json
{
  "intent": "SEARCH",
  "flow": "DISCOVERY"
}
```

------------------------------------------------------------------------

#### 3. Context Builder

Builds a **canonical execution context** that persists through the
request lifecycle.

This is the **spine object** of the system.

**Execution Context**

``` json
{
  "intent": "SEARCH",
  "flow": "DISCOVERY",
  "query": "...",
  "userId": "...",
  "sessionId": "...",
  "channel": "web"
}
```

------------------------------------------------------------------------

#### 4. Memory Manager

Selects **only relevant memory** for the current turn.

Memory types: - Session memory - User preferences - Semantic memory
(optional)

Memory is **pulled by the orchestrator**, never by agents.

**Memory Bundle**

``` json
{
  "sessionMemory": [],
  "userPreferences": null
}
```

------------------------------------------------------------------------

#### 5. Instruction Manager

This is one of the **most critical components**.

It decides: - How the agent should behave - What constraints apply -
What the agent is *not* allowed to do

Instruction sources: - System rules - Agent role definition -
Flow-specific behavior - Output constraints

**Instruction Bundle**

``` json
{
  "system": "...",
  "agent": "...",
  "flow": "...",
  "constraints": "JSON only"
}
```

------------------------------------------------------------------------

#### 6. Agent Dispatcher

Chooses **which agent** should handle the request.

Decision is based on: - Intent - Flow - Current orchestration state

**Output**

``` json
{
  "selectedAgent": "SearchAgent"
}
```

------------------------------------------------------------------------

## 5. Agent Contract (Universal)

All agents follow the **same input/output contract**.

### 5.1 Input to Any Agent

This is the **only boundary crossing** from orchestrator → agent.

``` json
{
  "instructions": { ...InstructionBundle },
  "context": { ...ExecutionContext },
  "memory": { ...MemoryBundle }
}
```

Agents: - Cannot see orchestrator internals - Cannot fetch memory -
Cannot alter instructions

------------------------------------------------------------------------

### 5.2 Agent Output

Agents always return **structured output**, never user-facing text
directly.

``` json
{
  "intent": "EXECUTE_SEARCH",
  "confidence": 0.92,
  "executionRequest": {
    "workflow": "NetworkSearch",
    "parameters": {
      "query": "bicycles",
      "filters": {}
    }
  }
}
```

------------------------------------------------------------------------

## 6. Guardrails Layer

Guardrails are applied **after agent reasoning** and **before
execution**.

They enforce: - Schema validity - Safety policies - Execution
eligibility - Cost / scope limits

Guardrails act as a **hard gate**.

------------------------------------------------------------------------

## 7. Workflow Execution Layer

If execution is required: - The orchestrator invokes a workflow -
Workflows interact with: - Networks - CDS - Databases - External APIs

Workflows may: - Call Payload Creation Agent (utility agent) - Emit
events to Kafka - Trigger async updates

------------------------------------------------------------------------

## 8. Feedback & Profiling Loop

Post-response: - User actions - Explicit feedback - Implicit signals

Are sent asynchronously to: - Feedback Agent - Profiling workflows

These updates: - Do not block user response - Improve future memory
selection

------------------------------------------------------------------------

## 9. End-to-End Flow (Concrete Example)

### User Query

> **"show me cool bicycles"**

### Step-by-step Flow

1.  Request Ingress normalizes input
2.  Intent Router → SEARCH / DISCOVERY
3.  Context Builder creates execution context
4.  Memory Manager selects (empty) memory
5.  Instruction Manager composes agent instructions
6.  Agent Dispatcher selects **Search Agent**
7.  Search Agent reasons and returns structured intent
8.  Guardrails validate output
9.  Workflow executes search
10. Results returned to user
11. Feedback captured asynchronously

[Mermaid](https://mermaid.live/edit#pako:eNp1lNtO4zAQhl_F8hVoC2pL6SESSGkK3UqElRKVale9McmQWiRx1wegIN59x4kTCnRz58w3439OfqOJSIF6VMFfA2UCM84yyYp1SfDbMql5wres1GRJmCJLBfK7aTG3tshGUHgqMwlKHcAiiy1KDXiIhNGHYgVTCwUCqRdNpobn6SEsDC0WQiHkjoSsZNlBYWF9o9LSJJqL8v-oP7Oon1ltM662TCebQ1zsWy4GJpONw49ubsLj7-S8SndumEwl4_mBgqwCS6yEfHzIxTMmLWTKS6bFgXsjv66x2opSAfGVguI-twprdnlyebmYe2RN1QZjFUASIXJyz5NdkoNa0wZczC0ZeeRWyILl_BVSIuvWOSByQJAzpfjDjvCqZa0VzcHUc428iK_8KPjZIdeYwsVsEQe_7q6i381twdTRVScJvEBiqkYkdYdbCrEw9JrGN-5h6P5fA_YDdebwZKtRVI1vGXeHG4d7U6Y5kCMotnp3vKfE5vVxB_nhHFxioTPXSvne0CjQLYOQP_P2Z0phoC-y_ZnDYhScaMKytn5oQlvs2xBP4hE-jVLjH_uOiYApFGAULzOCY9aa0T7HFsWVCCOxh7hPW9NGmEeOuMMWp0xDa7dWlmty1bbiFiCFtLY4X3ReBd4e82lC7LcKHBMZW6Aqh2c3yZ8oxCLfa9LEl8Hk2u0C5DjJt-Ljmq8arGO9Y_vqoUybNKOqEkucD9yb3Eav1mNd0g7NJE-phwWCDi0AZ90e6Zt1XFO9gQLW1O5LyuSjXY939MFV-yNE0bhJYbIN9R4YCu1Qs7WFdA9k-1eiHJCBMKWm3lm_VwWh3ht9od7JcHI6OT_vnndHvd6k3xuhdUe93mByOhwMz_vd3ll_3D3rD9879LW6t3s6snhvNO4NhpPJeDDuUGa0iHdl0qiClOMTEdbvdvV8v_8DUgDXfQ)

------------------------------------------------------------------------

## 10. Key Architectural Guarantees

-   Deterministic orchestration
-   Bounded agent behavior
-   Clear observability boundaries
-   LLM-agnostic design
-   Scalable across domains and networks
-   Safe-by-default execution

