# EXPLAINABILITY — OpenAI Agents SDK Runtime

> **Admissibility & Transparency Report for OpenGAP / Agent Passport**  
> *Agent Name:* OpenAI Agents SDK Runtime (`openai-agents-python`)  
> *Specification:* OpenGAP v0.1.0  
> *Domain:* Developer Tools / Agent Orchestration & Multi-Agent Systems  

---

## 1. Overview & Operational Purpose
The **OpenAI Agents SDK Runtime** provides an open-source, production-ready framework for building, running, and monitoring multi-agent systems in Python. It provides lightweight agent abstractions, first-class handoffs between specialized agents, robust Pydantic-powered tool definitions, and integrated guardrails.

Its operational purpose is to enable reliable, multi-agent collaboration where complex domains are segmented into focused, manageable agents that interact cleanly with tools, external APIs, and MCP servers.

---

## 2. How the Agent Decides (Decision-Making Logic)
OpenAI Agents SDK Runtime operates across a deterministic, multi-stage decision pipeline:

```
[Stage 1: Input Guardrail] ──> [Stage 2: Policy Routing] ──> [Stage 3: Function Execution]
                                                                       │
                                                                       ▼
[Stage 6: Trace Export] <── [Stage 5: Output Guardrail] <── [Stage 4: Handoff Evaluation]
```

### 2.1 Input Guardrail Inspection
- **Decision:** Evaluate incoming user message against registered input guardrails to identify prompt injections, prohibited content, or safety violations.
- **Rules:** If any guardrail raises a TripwireTriggered exception, abort model inference immediately and return the guardrail violation message.

### 2.2 Policy Routing & Model Invocation
- **Decision:** Send current conversation history, agent instructions, and available tool schemas to the model inference engine.
- **Rules:** The active agent persona and tool registry govern what capabilities and handoff functions are presented to the model.

### 2.3 Function Execution & Schema Validation
- **Decision:** Parse tool call requests from the model output and validate arguments against strict Pydantic models.
- **Rules:** Reject malformed parameters; execute valid tools within sandboxed callables and return structured results to the model context.

### 2.4 Handoff Evaluation & Output Guardrail
- **Decision:** Detect whether the model returned a handoff function to transfer execution to a subordinate agent, or generated a final response.
- **Rules:** If a handoff is triggered, switch active agent and continue loop; if a response is generated, pass through output guardrails before returning.

---

## 3. Data & Privacy
| Data Category | Retention Policy | Third-Party Sharing | Storage Mechanism |
|---|---|---|---|
| Conversation History & Turns | Ephemeral (Session Lifetime) | Model Provider Only | In-Memory Session State |
| Tool Invocation Arguments | Session Duration | External Tool APIs (if called) | In-Memory Execution Context |
| Pydantic Schema Definitions | Permanent (Static Code) | None | Local Python Files |
| OpenTelemetry Execution Traces | User Configured Retention | None | Local / Configured Trace Sink |

OpenAI Agents SDK Runtime complies with operational security and privacy standards:
- **No Cloud Data Exfiltration:** Model inferences and tool calls communicate only with explicitly configured endpoints; no ambient telemetry is sent to unauthorized third parties.
- **Epistemic Isolation:** Each agent in a multi-agent hierarchy maintains isolated local context; handoffs share only explicitly designated state variables.
- **Sanitized Model Payloads:** Private credential keys and environmental configurations are filtered from model context windows and error strings.
- **Data Minimization:** Only conversation history, instructions, and tool definitions essential for the current turn are dispatched in inference payloads.

---

## 4. Known Limitations & Failure Modes
Reviewers, auditors, and users should note the following operational constraints:
1. Cyclic Agent Handoff Loops
   - *Limitation:* Poorly constrained agent transfer definitions can result in agents repeatedly delegating tasks to one another indefinitely.
   - *Mitigation:* The runtime enforces a hard max_handoffs limit (default: 10), after which execution terminates with a recursion exception.
2. Pydantic Schema Parsing Discrepancies
   - *Limitation:* Highly complex nested recursive schemas may occasionally produce invalid JSON Schema definitions or parsing errors in smaller models.
   - *Mitigation:* The runtime validates schema generation upfront and provides automatic validation retry loops.
3. Network Latency in Remote MCP Tools
   - *Limitation:* External tools hosted on high-latency MCP servers can delay overall agent response times.
   - *Mitigation:* The runtime supports async tool execution and enforces strict per-tool timeouts.
4. Token Window Exhaustion across Deep Handoff Chains
   - *Limitation:* Accumulating extensive conversation history across multiple agent transitions can exhaust model context windows.
   - *Mitigation:* The runtime supports turn truncation, context summarization, and selective state handoffs.

---

## 5. Verification, Safety & Human Oversight
OpenAI Agents SDK Runtime integrates multi-layer safety rails to ensure full human accountability and system integrity:
- **Real-Time Human Approval Gate:** Tools designated as sensitive can be configured to require explicit human confirmation before execution.
- **Emergency Session Interrupt:** Long-running multi-agent execution loops can be cancelled immediately via Python `asyncio.CancelledError` or cancellation tokens.
- **Step Quota Guardrails:** Strict iteration caps (`max_turns`) prevent runaway model loops and excessive token expenditures.
- **Structured Audit Logging:** Every agent decision, tool execution, handoff, and guardrail check is recorded in structured, immutable trace events.
