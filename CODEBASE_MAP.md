<!-- Last verified: 2026-03-21 by /codebase-mirror -->

# AEE (Agent Envelope Exchange) — Codebase Map

## Metrics

| Metric | Count |
|--------|-------|
| Spec files | 3 (aee.md, intents.md, quickstart.md) |
| Schemas | 1 (decision-evidence.schema.json) |
| Supporting docs | 4 (relationship-to-mcp-acp.md, examples/, CHANGELOG, README) |
| GitHub templates | 3 (bug, feature, PR template) |

## Summary

AEE defines the ULID-based message envelope protocol for agent communication. Envelopes carry `from`, `to`, `intent`, `corr` (conversation), and `reply_to` fields for causality chains.

## Key Files

- aee.md — Full specification
- intents.md — Intent taxonomy
- quickstart.md — Getting started guide
- schemas/decision-evidence.schema.json — Decision evidence JSON schema
- relationship-to-mcp-acp.md — Comparison with MCP/ACP protocols
- AI_README.json — Machine-readable project summary
