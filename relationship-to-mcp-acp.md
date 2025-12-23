# Relationship to MCP and ACP

AEE (Agent Envelope Exchange) is intentionally narrow in scope.

It standardizes a **message envelope** for agent-to-agent communication, focusing on:
- identity (`from`, `to`)
- intent
- correlation and tracing
- minimal execution constraints

AEE does **not** define:
- how tools are exposed
- how agents discover each other
- how workflows are orchestrated
- how messages are transported

## Relationship to MCP (Model Context Protocol)

MCP focuses on standardizing how AI applications connect to **tools and context providers**.

AEE can be used **alongside MCP** by:
- invoking MCP tools to perform work
- packaging tool results into AEE messages
- routing those messages between agents or workflow stages

In this model:
- MCP handles *capability exposure*
- AEE handles *inter-agent message structure*

## Relationship to ACP (Agent Communication Protocol)

ACP aims to enable **interoperable agent-to-agent communication** via standardized APIs.

AEE does not compete with ACP and can act as:
- an internal message format
- a wire payload carried over ACP endpoints
- a common envelope inside ACP-based systems

In this model:
- ACP defines *where and how messages are delivered*
- AEE defines *what a message looks like once delivered*

## Middleware Adoption Strategy

The fastest path to AEE adoption is **middleware injection at the LLM gateway layer**.

### How It Works

A proxy layer (e.g., LiteLLM, Portkey, or a custom gateway) intercepts LLM requests and responses, wrapping them in AEE envelopes automatically:

1. **Request:** User prompt → Gateway wraps in MVE envelope → LLM
2. **Response:** LLM output → Gateway wraps in MVE envelope → Logs/downstream

Developers change **no application code**. They get:
- Structured, correlatable logs
- Consistent envelope format across heterogeneous LLM calls
- Observability without instrumentation

### What Middleware Does

- Emits MVE-Required or MVE-5 envelopes (depending on configuration)
- Generates `id`, `ts`, `corr` automatically
- Populates `from` (gateway identifier) and `intent` (e.g., `llm.completion.request`)
- Preserves prompt/response in `payload`

### What Middleware Does NOT Promise

- **Semantic correctness:** The `intent` is generic (`llm.completion.*`), not domain-specific
- **Payload schema compliance:** The payload is the raw LLM request/response
- **Full AEE protocol support:** Middleware does not implement `aee.*` intents by default

Middleware is the **wedge**: it gets AEE envelopes into production systems without requiring buy-in. Once teams see the observability benefits, they adopt deeper integration.

## Summary

MCP, ACP, and AEE operate at different layers:

- MCP: tools and context
- ACP: agent connectivity
- AEE: message envelopes

They are complementary, not mutually exclusive.
