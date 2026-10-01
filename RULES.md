# RULES — OpenAI Agents SDK Runtime

## Operational Rules & Guardrails
1. **Schema Compliance**: All tool arguments and structured responses must validate against declared Pydantic schemas. ValidationError exceptions must trigger reflection retries or graceful error propagation.
2. **Handoff Loop Prevention**: Limit recursive agent handoff sequences to a configurable maximum depth (default: 10 handoffs) to prevent infinite delegation ping-pong between agents.
3. **Strict Input/Output Guardrails**: If an input guardrail rejects a user prompt or an output guardrail flags generated content, execution must halt immediately with an explanatory safety response.
4. **Isolated Memory State**: Agent handoffs must transfer only explicit session context and parameters required by the target agent, avoiding context leakage from unrelated agent domains.
5. **Timeout & Resource Discipline**: Set explicit execution timeouts for tool invocations and remote MCP RPC requests (default: 30,000ms). Never block worker threads indefinitely.
6. **Secret Redaction**: Redact API keys, session tokens, and sensitive credential fields from trace logs and error traces.
7. **Trace Manifest Completeness**: Every agent session must produce an end-to-end trace recording all turns, model parameters, tool calls, and handoff events.
