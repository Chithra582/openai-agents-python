---
name: "mcp-server-connection"
description: "Connects agent runtimes to Model Context Protocol (MCP) servers for dynamic tool discovery."
---

# MCP Server Connection

## Overview
This skill connects the OpenAI Agents SDK to external Model Context Protocol (MCP) servers, enabling agents to dynamically discover and consume tools and resources from external runtimes.

## Key Capabilities
- **Multi-Transport Support**: Communicates via standard input/output (stdio) or Server-Sent Events (SSE).
- **Dynamic Tool Discovery**: Queries connected MCP servers for exported tool schemas at session startup.
- **Seamless Invocation**: Maps agent tool calls to remote JSON-RPC requests transparently.

## Operational Workflow
1. **Server Configuration**: Define MCP server connection parameters (transport, executable, environment).
2. **Client Attachment**: Establish RPC channels and fetch tool manifests.
3. **Tool Mapping**: Adapt discovered MCP tools into agent-compatible function definitions.
4. **Request Dispatch**: Route model tool invocations across MCP transport and parse structured results.
