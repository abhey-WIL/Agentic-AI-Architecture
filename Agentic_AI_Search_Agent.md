# Search Agent – Architecture & Execution Flow

---

## 1. Purpose of the Search Agent

The **Search Agent** is responsible for interpreting discovery queries and deciding **how a search should be executed** across networks and domains.

It does not directly execute searches.  
It plans and selects the optimal search strategy based on:
- User profiling
- Historical performance data
- Cached domain–network intelligence
- Query intent and ambiguity

The Search Agent outputs a **structured search plan** that is later executed by workflows.

---

## 2. What the Search Agent Is (and Is Not)

### The Search Agent is
- An LLM-powered reasoning agent
- A planner and decision-maker
- A selector of search strategies
- A consumer of curated memory

### The Search Agent is not
- A workflow
- A network executor
- A catalog fetcher
- A state owner

---

## 3. High-Level Responsibilities

The Search Agent determines:
1. What the user is looking for
2. Which domains and networks are relevant
3. Whether sufficient prior knowledge exists
4. Which search strategy to apply
5. How a search payload should be constructed

---

## 4. Inputs to the Search Agent

The Search Agent receives a bounded input from the Base Orchestrator.

### 4.1 Agent Input Contract
```json
{
  "instructions": { ... },
  "context": {
    "query": "show me cool bicycles",
    "intent": "SEARCH",
    "flow": "DISCOVERY",
    "userId": "...",
    "sessionId": "..."
  },
  "memory": {
    "userProfile": { ... },
    "networkDomainCache": { ... },
    "searchHistory": { ... }
  }
}
```

---

## 5. Memory Consumed by the Search Agent

Memory is selected upstream by the orchestrator.  
The Search Agent only consumes it.

### 5.1 User Profiling Memory

Contains:
- Preferred domains
- Preferred networks
- Past transaction patterns
- Implicit preferences
```json
{
  "preferredDomains": ["mobility"],
  "preferredNetworks": ["network-A"],
  "frequentCategories": ["bicycles"]
}
```

### 5.2 Network & Domain Intelligence Cache

System-owned intelligence accumulated over time.

Contains:
- Domain ↔ network mappings
- Historical success scores
- Relevance indicators
```json
{
  "bicycles": {
    "networks": [
      { "id": "network-A", "successScore": 0.82 },
      { "id": "network-C", "successScore": 0.45 }
    ]
  }
}
```

---

## 6. Internal Structure of the Search Agent

Conceptually, the Search Agent comprises:
- Instruction Set
- Reasoning Core (LLM)
- Strategy Selection Logic
- Payload Planning Logic
- Beckn Network Tool Usage
- Structured Output Formatter

### 6.1 Instruction Set

Defines:
- Agent role
- Constraints
- Allowed decisions
- Output schema

### 6.2 Reasoning Core (LLM)

Used for:
- Interpreting natural language queries
- Mapping queries to domains and categories
- Evaluating confidence in available data

### 6.3 Strategy Selection

Determines which search path to take:

- Experience-based CDS search
- Natural language CDS catalog search

Primary decision:
- Do we have sufficient prior knowledge to run a targeted search?

---

## 7. Search Strategy Paths

### 7.1 Experience-Based CDS Search

**Used when:**
- Historical data exists
- High-confidence domain–network mappings are available

**Flow:**
1. Identify domain and category
2. Select high-confidence networks
3. Plan a structured CDS search
4. Generate a protocol-compliant payload

### 7.2 Natural Language CDS Search

**Used when:**
- Data is insufficient or unavailable
- Cold-start or unfamiliar domain scenarios

**Flow:**
1. Construct a natural language search
2. Query CDS catalog endpoints
3. Capture results for learning

---
## 8. Beckn Payload Generation Agent

The Beckn Payload Generation Agent is responsible for constructing protocol-compliant Beckn payloads based on structured parameters provided by business agents such as the Search Agent.

The Search Agent does **not** generate Beckn payloads directly.  
Instead, it determines **what needs to be searched** and provides the necessary parameters required to construct the payload.

---

### 8.1 Responsibilities of the Payload Generation Agent

The Payload Generation Agent:
- Translates high-level intent into Beckn protocol structures
- Populates mandatory and optional Beckn fields
- Ensures schema correctness and protocol compliance
- Manages context fields such as domain, action, transaction IDs, and metadata
- Abstracts Beckn protocol complexity from business agents

---

### 8.2 Inputs from the Search Agent

The Search Agent provides a structured parameter set, such as:

```json
{
  "action": "search",
  "domain": "mobility",
  "category": "bicycles",
  "itemQuery": "cool bicycles",
  "networks": ["network-A"],
  "fulfillmentType": "Delivery"
}
```

---

## 9. Downstream Flow

1. Output validation and guardrails
2. Workflow executes search
3. CDS results returned
4. Results cached
5. Feedback signals recorded

---

## 10. Feedback & Learning Loop

**Post-search signals:**
- Clicks
- Conversions
- Empty results
- Latency

**These update:**
- User profiling
- Network and domain intelligence
- Strategy confidence scores

---

## 11. End-to-End Flow Summary

1. Query reaches Search Agent
2. Relevant memory is consumed
3. LLM reasons over intent and confidence
4. Search strategy selected
5. Payload planning performed
6. Structured output returned
7. Workflow executes search
8. Feedback updates intelligence stores

---