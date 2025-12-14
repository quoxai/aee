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

## Summary

MCP, ACP, and AEE operate at different layers:

- MCP: tools and context
- ACP: agent connectivity
- AEE: message envelopes

They are complementary, not mutually exclusive.
