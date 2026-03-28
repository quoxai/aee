<!-- Last verified: 2026-03-27 by /codebase-mirror -->

# AEE (Agent Envelope Exchange) — Codebase Map

## Spec Status
| Field | Value |
|-------|-------|
| Version | 1 (stable) |
| IETF Draft | draft-cowles-aee-00 |
| License | MIT |

## Envelope Structure
14-field JSON envelope for agent-to-agent communication.
- **Required (10):** v, id, ts, type, from, to, intent, corr, priority, payload
- **Optional (4):** reply_to, trace, requires, sig
- **Types:** task, result, event, error, stream
- **Reserved namespace:** `aee.*`

## Key Files
| File | Purpose |
|------|---------|
| `README.md` | Human-readable spec (326 lines) |
| `AI_README.json` | Full spec as self-contained AEE envelope |
| `schemas/decision-evidence.schema.json` | Evidence fragment schema (JSON Schema Draft 2020-12) |
| `CHANGELOG.md` | Version history |
