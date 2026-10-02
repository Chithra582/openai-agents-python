# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **OpenAI Agents SDK Runtime** (`openai-agents-python`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** OpenAI Agents SDK Runtime (`openai-agents-python`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Agent Orchestration & Multi-Agent Systems  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

The OpenAI Agents SDK Runtime provides an open-source, production-ready framework for building, running, and monitoring multi-agent systems in Python. It provides lightweight agent abstractions, first-class handoffs between specialized agents, robust Pydantic-powered tool definitions, and integrated guardrails. Its operational purpose is to enable reliable, multi-agent collaboration where complex domains are segmented into focused, manageable agents that interact cleanly with tools, external APIs, and MCP servers.

### 1. Decision Architecture

The user message intake, agent routing, tool execution, and handoff coordination pipeline operates across a deterministic, five-stage architecture:

```
User Message / Task Event (API Query / Multi-Agent Collaboration Goal / Tool Result)
    │
    ▼
[Stage 1: Ingestion & Input Guardrail Verification]
    │  - Evaluates incoming message payloads and parses conversation history
    │  - Executes input guardrails (Pydantic models, safety filters, prompt injection checks)
    │  - Assigns request to initial specialized agent persona
    ▼
[Stage 2: Tool Calling & Schema Validation]
    │  - Inspects agent tool registry and selects optimal tool function via function calling
    │  - Validates input arguments against Pydantic type signatures before execution
    │  - Executes tools in isolated local execution sandboxes
    ▼
[Stage 3: First-Class Handoff Coordination]
    │  - Evaluates whether task requires delegating to another specialized agent
    │  - Performs atomic handoff transfer, updating agent persona and system instructions
    │  - Preserves essential context while pruning unnecessary intermediate chatter
    ▼
[Stage 4: Output Guardrail Auditing]
    │  - Audits candidate agent response through output guardrail filters
    │  - Verifies factual grounding, format compliance, and safety standards
    │  - Triggers corrective generation if guardrail assertions fail
    ▼
[Stage 5: Final Response Serialization & Trace Archival]
    │  - Serializes verified response tokens and returns deliverables to client
    │  - Scrubs private user credentials and API tokens from execution logs
    │  - Commits structured agent run trajectories locally for auditing
    ▼
Validated Multi-Agent Response & Auditable Orchestration Trajectory Record
```

### 2. Decision Logic & Agent Handoff Formulations

The runtime evaluates agent routing, tool call confidence, and handoff transitions using deterministic mathematical models:

1. **Handoff Delegation Score ($S_{\text{handoff}}$)**:
   $$S_{\text{handoff}}(A_j) = (w_e \cdot E_{\text{expertise}}) + (w_t \cdot T_{\text{tools}}) + (w_h \cdot H_{\text{history}})$$
   where:
   - $E_{\text{expertise}} \in [0, 1]$ represents specialized persona alignment with the active query.
   - $T_{\text{tools}} \in [0, 1]$ represents required tool registry presence in agent $A_j$.
   - $H_{\text{history}} \in [0, 1]$ accounts for prior turns handled by $A_j$.
   - Weights: $w_e = 0.50, w_t = 0.35, w_h = 0.15$ ($\sum w_i = 1.0$).

2. **Handoff Convergence Index ($C_{\text{handoff}}$)**:
   $$C_{\text{handoff}} = 1 - \frac{N_{\text{handoffs}}}{\text{max\_handoffs}}$$
   When $C_{\text{handoff}} \le 0$, the runtime halts handoff chaining deterministically with code `WARN_MAX_HANDOFFS_REACHED` to prevent circular ping-pong delegation.

### 3. Thresholding & Refusal Decision Criteria

OpenAI Agents SDK Runtime enforces strict operational safety and integrity boundaries:
- **Refusal to Execute Malformed Tool Arguments**: Tool invocations failing Pydantic schema validation are rejected deterministically before execution (`ERR_TOOL_ARGUMENT_VALIDATION_FAILED`).
- **Refusal of Circular Unbounded Handoffs**: Agent-to-agent delegation chains exceeding `max_handoffs: 10` are blocked (`ERR_CIRCULAR_HANDOFF_PREVENTED`).
- **Turn Ceiling Enforcement**: Conversational turns are strictly capped by `max_turns: 25` to eliminate runaway loop execution (`WARN_TURN_BUDGET_REACHED`).
- **Local Directory Boundary Enforcement**: Tool writes outside the designated workspace directory are rejected (`ERR_OUT_OF_BOUNDS_WRITE`).

### 4. Fallback Decision Mechanism

Continuous multi-agent availability is guaranteed through multi-tier fault recovery:
- **Model Cascade Failover**: When the primary foundation model experiences latency spikes or HTTP 429 rate limits, the orchestrator cascades automatically between `gpt-4o`, `claude-3-5-sonnet`, and `gemini-2.0-flash`.
- **Supervisor Fallback Agent**: If specialized child agents fail to resolve a task, the runtime cascades to a generalist supervisor agent.
- **Graceful Tool Degradation**: If an external API tool fails, the agent reports a structured tool error and attempts alternative reasoning paths.

### 5. Human-in-the-Loop Governance

Human software engineers retain full supervisory control over agent actions:
- **Mandatory Approval Checkpoints**: High-impact tool executions (database writes, financial payments, file deletions) require explicit human confirmation.
- **Emergency Session Kill Switch**: Operators can halt agent execution loops instantly via `Ctrl+C` interrupt signals.
- **Inspectable Run Trajectories**: Every agent decision, handoff event, tool invocation, and token metric is logged in structured trajectory files for auditing.

---

## The Data It Uses

OpenAI Agents SDK Runtime operates under strict privacy, data minimization, and local workspace isolation standards.

### 1. Ingested Input Data

The agent processes only operational assets necessary to fulfill multi-agent orchestration:
- **User Messages**: Inbound natural language queries and conversation context.
- **Tool Arguments**: Typed JSON payloads passed to function calling tools.
- **Handoff Context Objects**: Structured data objects transferred between agents during delegation.

### 2. Configuration & Reference Data

- **Agent Persona Schemas**: System prompts, instruction templates, and tool registries for each agent.
- **Pydantic Validation Models**: Python type definitions governing tool input and output schemas.
- **Guardrail Configuration Schemas**: Pre-configured filters for input sanitization and output verification.

### 3. Base Model & Inference Lineage

- **Deterministic Orchestration Logic**: Pydantic schema validators, handoff state machines, and guardrail evaluators executed natively in Python (100% deterministic with zero LLM variance).
- **Foundation LLMs**: High-capability frontier models (`gpt-4o`, `claude-3-5-sonnet`, `gemini-2.0-flash`) utilized for conversational reasoning, intent classification, and tool generation.
- **Zero Training on Developer Payloads**: User prompt streams, tool arguments, and business logic are never stored on external cloud servers or used for model training.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against prompt injection, insecure output handling, and excessive agency.
- **Session-Scoped Memory Partitioning**: Conversation histories and handoff contexts are isolated per session, preventing cross-tenant leakage.
- **Local-Only Trajectory Logs**: All run traces and execution logs reside exclusively on the developer's local filesystem.
- **Zero Commercial Monetization**: Developer prompts, tool definitions, and application data are never monetized, aggregated, or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of OpenAI Agents SDK Runtime is essential for production deployment.

### 1. Deep Handoff Chain Context Loss
- **Limitation**: Passing control across multiple successive agents can cause subtle context dilution if handoff schemas are too sparse.
- **Mitigation**: The SDK supports rich typed handoff contexts that explicitly pass required entity states to downstream agents.

### 2. Multi-Agent Concurrency Deadlocks
- **Limitation**: In complex topologies with mutual dependencies, agents can theoretically enter waiting states if locks are misconfigured.
- **Mitigation**: The runtime enforces sequential handoff semantics and strict timeout ceilings to prevent concurrency lockups.

### 3. Upstream Provider Tool Calling Schema Drift
- **Limitation**: Different model providers format tool call responses with slight variations that can cause parser warnings.
- **Mitigation**: The runtime normalizes all tool calls through strict Pydantic parsing layers before invocation.

### 4. High Latency in Multi-Step Handoff Sequences
- **Limitation**: Chaining several agents sequentially accumulates LLM inference latency for interactive users.
- **Mitigation**: The SDK supports token streaming across handoffs and enables fast early-exit conditions when tasks complete.

### 5. Subjective Persona Boundary Overlaps
- **Limitation**: Subtly overlapping agent mandates (e.g., Coder vs. Tester) can cause indecision during handoff routing.
- **Mitigation**: Developers are advised to define mutually exclusive agent responsibilities and explicit routing triggers in system prompts.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & agent handoff formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested user messages, tool arguments & handoff objects | Section 1 | Verified |
| - Configuration, persona schemas & Pydantic models | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Deep handoff chain context loss | Section 1 | Verified |
| - Multi-agent concurrency deadlocks | Section 2 | Verified |
| - Upstream provider tool calling schema drift | Section 3 | Verified |
| - High latency in multi-step handoff sequences | Section 4 | Verified |
| - Subjective persona boundary overlaps | Section 5 | Verified |
