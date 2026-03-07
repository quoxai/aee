<!-- Last verified: 2026-03-06 by /codebase-mirror -->

# AEE (Agent Envelope Exchange) — Codebase Map

## Metrics
| Metric | Count |
|--------|-------|
| Spec Files | 7 markdown |
| IETF Status | Submitted (draft-cowles-aee-00) |
| Envelope Fields | 14 (10 required, 4 optional) |
| Message Types | 5 |

## Key Specs
- **Format:** 14-field JSON envelope
- **Fields:** v, id, ts, type, from, to, intent, corr, priority, payload + reply_to, trace, requires, sig
- **Types:** task, result, event, error, stream
- **Causality:** corr (shared conversation), reply_to (direct lineage)
- **Reserved intents:** aee.status.ping, aee.capability.list, aee.context.fetch

## Files
| File | Purpose |
|------|---------|
| README.md | Protocol overview |
| aee.md | Full specification |
| intents.md | Intent registry |
| quickstart.md | Getting started |
| relationship-to-mcp-acp.md | Protocol positioning |
| examples/handshake.md | Agent discovery |
| AI_README.json | Self-referential AEE envelope |
