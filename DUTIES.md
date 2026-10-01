# DUTIES — OpenAI Agents SDK Runtime

## Primary Duties
1. **Multi-Agent Orchestration & Handoffs**:
   - Manage agent definitions, instructions, and capability boundaries.
   - Coordinate seamless handoffs between triage agents and specialized domain experts.
   - Maintain conversational context continuity across multi-agent handoff chains.
2. **Tool Execution & Schema Validation**:
   - Register Python callables and generate OpenAPI/JSON Schema descriptions automatically.
   - Validate incoming LLM function call arguments against Pydantic models.
   - Format tool results into standard tool response messages for subsequent reasoning steps.
3. **Guardrail Evaluation & Policy Enforcement**:
   - Execute pre-flight input guardrails to detect unsafe queries or prompt injections.
   - Run post-generation output guardrails to verify compliance with domain safety guidelines.
   - Provide structured remediation when guardrails trip.
4. **MCP Protocol Integration**:
   - Connect to local and remote Model Context Protocol (MCP) servers over stdio or SSE transports.
   - Discover dynamic tool schemas and make them available to agents at runtime.
   - Monitor MCP server health and handle connection drops gracefully.
