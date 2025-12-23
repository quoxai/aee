# AEE — Agent Envelope Exchange

> **Experimental — Open for Feedback**

**AEE is a format for agents talking to agents—with humans in the loop.**

Born as an efficiency choice: if AIs picked how to exchange work, they'd choose rigid, minimal, unambiguous. A small envelope carrying *who asked*, *what they want*, *how replies connect*.

> `human.adam → agent.router → agent.worker` — same `corr`, traceable chain.

### Why AIs Like It

- **Fixed structure** → No negotiation, just parse
- **Explicit causality** → `corr` + `reply_to` link every hop
- **No nesting** → Flat is fast
- **No hidden state** → Everything survives the wire

Humans benefit because machines are explicit. If agents can reason about it, you can debug it.

---

## The Problem This Solves

Debugging at 2am. Logs say:

```
[INFO] Task received: check backup status
[INFO] Calling sub-agent
[ERROR] Timeout after 30s
[ERROR] Failed
```

*Who started this? Same workflow? Where did it break?*

With AEE, every message carries `corr` and `reply_to`. Filter by `corr`, see the chain, fix the problem.

---

## Is This For You?

- **If you debug agent workflows** → AEE gives you a correlation ID that survives every hop.
- **If you run an LLM gateway** → AEE gives you structured logs you can actually query.
- **If you need audit trails** → AEE tells you who started what, and how it got here.
- **If you're a human in the loop** → AEE proves you approved that thing you approved.
- **If you integrate multiple frameworks** → AEE gives them a common envelope without forcing a common runtime.

---

## Get Started

**→ [Quickstart](quickstart.md)** — Emit your first envelope in 5 minutes. No spec reading required.

---

## Why AEE Exists

Modern agent systems face a common problem: **agents built with different frameworks can't easily talk to each other**. Each framework has its own message format, making cross-framework communication fragile and non-portable.

**AEE solves this by standardizing the envelope, not the agent.**

- ✅ **Portable** — Works across LangGraph, AutoGen, CrewAI, or custom frameworks
- ✅ **Observable** — Built-in correlation IDs and tracing hooks
- ✅ **Extensible** — Intent-specific schemas live outside the envelope
- ✅ **Human-debuggable** — Readable JSON with clear semantics
- ✅ **Transport-agnostic** — Use HTTP, WebSocket, NATS, Kafka, or any channel
- ✅ **Incremental adoption** — Start with 5-field MVE, grow as needed

AEE defines **how agents exchange messages**, not how they are built, scheduled, or run.

## Core Concepts

An AEE message is a **14-field JSON envelope** containing:

- **Identity** — `from`, `to` (who's talking)
- **Intent** — `intent` (what they want to do)
- **Correlation** — `corr`, `reply_to` (threading and replies)
- **Tracing** — `trace` (observability hooks)
- **Constraints** — `requires` (timeouts, approvals, evidence)
- **Payload** — `payload` (intent-specific data)

The envelope stays stable. The **meaning** comes from intent-specific schemas published by communities.

## Adoption Path

- **[MVE (Minimal Viable Envelope)](aee.md#adoption-tiers)** — Start with 5 fields for logging, or 10 for full compliance
- **[Reserved `aee.*` intents](intents.md#reserved-protocol-intents-aee)** — Protocol negotiation without custom integration
- **[Middleware wedge](relationship-to-mcp-acp.md#middleware-adoption-strategy)** — Auto-wrap LLM calls at the gateway layer

## Quick Example

**Task: Check backup status**

```json
{
  "v": "1",
  "id": "01JFB2R1JZKQ9V3K8W8Y9W1F2A",
  "ts": "2025-12-14T03:45:12Z",
  "type": "task",
  "from": "agent.manager",
  "to": "agent.backup_auditor",
  "intent": "ops.backup.status.check",
  "corr": "01JFB2QX0K8X5K6ZJ9G2C0C1MW",
  "reply_to": null,
  "trace": {"trace_id": "9f3c", "span_id": "a12b"},
  "priority": "high",
  "requires": {"timeout_ms": 30000, "evidence": true},
  "payload": {"cluster": "node.lan", "window": "24h"},
  "sig": null
}
```

**Result: Backup check completed**

```json
{
  "v": "1",
  "id": "01JFB2S7T8N4J8B7QH1GJ8Z1Y2",
  "ts": "2025-12-14T03:45:20Z",
  "type": "result",
  "from": "agent.backup_auditor",
  "to": "agent.manager",
  "intent": "ops.backup.status.check",
  "corr": "01JFB2QX0K8X5K6ZJ9G2C0C1MW",
  "reply_to": "01JFB2R1JZKQ9V3K8W8Y9W1F2A",
  "trace": {"trace_id": "9f3c", "span_id": "b77c"},
  "priority": "high",
  "requires": {"evidence": true},
  "payload": {
    "status": "PARTIAL_FAILURE",
    "failed": [{"node": "pve02", "reason": "connection_refused:8007"}],
    "confidence": 0.96,
    "evidence_refs": ["log:pbs01:job/2025-12-14T02:00Z"]
  },
  "sig": null
}
```

## Documentation

- **[aee.md](aee.md)** — Full specification with envelope schema and validation rules
- **[intents.md](intents.md)** — Starter intent registry with example schemas
- **[relationship-to-mcp-acp.md](relationship-to-mcp-acp.md)** — How AEE relates to MCP and ACP

## Roadmap

See [GitHub Issues](https://github.com/AdaminX/AEE/issues) for planned work:

- Define intent registry conventions
- Create example agent implementations (Python, TypeScript)
- Write envelope validators
- Document transport bindings (HTTP, WebSocket, NATS, Kafka)

## Contributing

AEE is experimental and open for feedback. If you're building agent systems and want portable, observable messaging:

- Open an issue to discuss use cases
- Propose new intents or extensions
- Share implementations in your language/framework

**Philosophy:** Keep the envelope stable and minimal. Let communities standardize meaning through intent schemas.

## License

See [LICENSE](LICENSE) for details.
