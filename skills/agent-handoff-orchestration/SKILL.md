---
name: "agent-handoff-orchestration"
description: "Coordinates dynamic multi-agent handoffs and task transfers across specialized agent workflows."
---

# Agent Handoff Orchestration

## Overview
This skill coordinates multi-agent architectures where control of the conversation transfers dynamically from one specialized agent to another using explicit handoff functions.

## Architectural Model
- **Triage Agent**: Understands high-level user intent and delegates to specialized sub-agents.
- **Specialized Agents**: Domain-specific agents equipped with focused toolsets (e.g., Billing, Support, Analytics).
- **Handoff Functions**: Registered callables that transfer conversation authority and context state.

## Operational Workflow
1. **Agent Setup**: Configure agents with distinct system instructions and relevant handoff functions.
2. **Handoff Detection**: The active agent evaluates conversation history and invokes a handoff function when appropriate.
3. **Context Transition**: The runtime switches active agent identity, preserving conversation context.
4. **Execution Continuation**: The receiving agent takes the next turn, generating response or calling local tools.
