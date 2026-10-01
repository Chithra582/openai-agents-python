---
name: "agent-guardrail-enforcement"
description: "Applies input and output safety guardrails to enforce conversational and policy boundaries."
---

# Agent Guardrail Enforcement

## Overview
This skill provides programmatic verification checks on user inputs and agent responses to ensure safety, policy compliance, and predictable behavior.

## Guardrail Types
- **Input Guardrails**: Screen user prompts before they reach model inference (e.g., detecting prompt injection, profanity, or out-of-scope requests).
- **Output Guardrails**: Verify generated model responses before returning to users (e.g., verifying factual constraints or preventing PII leaks).

## Operational Workflow
1. **Guardrail Registration**: Attach custom guardrail functions to the agent configuration.
2. **Pre-Inference Evaluation**: Execute input guardrails sequentially against incoming user messages.
3. **Tripwire Handling**: If a guardrail triggers, intercept flow and return predefined fallback responses.
4. **Post-Inference Verification**: Validate generated text through output guardrails before returning final output.
