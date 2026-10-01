---
name: "pydantic-tool-validation"
description: "Validates function calling arguments and structured outputs using strict Pydantic schemas."
---

# Pydantic Tool Validation

## Overview
This skill guarantees that all function calling tools and structured LLM outputs are governed by strict Pydantic v2 data models, eliminating type mismatches and runtime parameter errors.

## Key Capabilities
- **Schema Introspection**: Automatically converts Python type annotations and Pydantic models into standard JSON Schemas.
- **Runtime Validation**: Validates raw model JSON arguments against expected field types and bounds.
- **Structured Error Feedback**: Formats validation errors into actionable error messages for model self-correction.

## Operational Workflow
1. **Function Registration**: Register Python functions with Pydantic type signatures.
2. **Schema Generation**: Export tool schemas into OpenAI-compatible tool specifications.
3. **Argument Parsing**: Validate incoming arguments through Pydantic model validation.
4. **Invocation**: Pass validated model instances into the target Python callable.
