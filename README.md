# AEE — Agent Envelope Exchange

> **Status:** Experimental — Open for Feedback
> **Version:** 1
> **License:** MIT

**IETF Internet-Draft:** [`draft-cowles-aee-00`](https://datatracker.ietf.org/doc/draft-cowles-aee/)

**AEE is a format for agents talking to agents—with humans in the loop.**

Born as an efficiency choice: if AIs picked how to exchange work, they'd choose rigid, minimal, unambiguous. A small envelope carrying *who asked*, *what they want*, *how replies connect*.

### Why AIs Like It

- **Fixed structure** → No negotiation, just parse
- **Explicit causality** → `corr` + `reply_to` link every hop
- **No nesting** → Flat is fast
- **No hidden state** → Everything survives the wire

Humans benefit because machines are explicit. If agents can reason about it, you can debug it.

### How It Flows

```
human.adam                       # A human starts a task
    │
    │ id: aaa-111
    │ corr: xyz-789
    │ intent: backup.check
    ▼
agent.router                     # Router delegates
    │
    │ id: bbb-222
    │ corr: xyz-789              # same corr
    │ reply_to: aaa-111          # points back
    ▼
agent.worker                     # Worker does the job
    │
    │ id: ccc-333
    │ corr: xyz-789              # same corr
    │ reply_to: bbb-222          # points back
    ▼
result (or error)                # Response bubbles up
    │
    │ id: ddd-444
    │ corr: xyz-789              # same corr
    │ reply_to: ccc-333          # traceable chain
    ▼
human.adam sees the result
```

Every hop shares `corr`. Every response carries `reply_to`. The chain is traceable from human to final result.

### Mental Model

- **AEE is like HTTP headers** — it standardizes the envelope, not the content
- **AEE is not orchestration** — it doesn't schedule, retry, or route
- **AEE is not a runtime** — it doesn't execute anything
- **AEE is not a framework** — it doesn't care how your agents are built
- **AEE is just structure** — 14 fields that make causality explicit

### `type` vs `intent`

| Field | What it is | Examples |
|-------|-----------|----------|
| `type` | Envelope category (fixed set) | `task`, `result`, `event`, `error`, `stream` |
| `intent` | Payload meaning (your namespace) | `ops.backup.check`, `aee.capability.list` |

`type` tells you *how* to handle the envelope. `intent` tells you *what* it means.

### Mini Example (Copy/Paste)

Two envelopes. Same `corr`. The `result` points back with `reply_to`.

**Task:**
```json
{
  "v": "1",
  "id": "01EXAMPLE_TASK_0000000000001",
  "ts": "2025-12-23T12:00:00Z",
  "type": "task",
  "from": "human.adam",
  "to": "agent.router",
  "intent": "demo.hello.world",
  "corr": "01EXAMPLE_CORR_0000000000001",
  "reply_to": null,
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {"message": "Hello from a human. Route this to a worker."},
  "sig": null
}
```

**Result:**
```json
{
  "v": "1",
  "id": "01EXAMPLE_RESULT_000000000001",
  "ts": "2025-12-23T12:00:02Z",
  "type": "result",
  "from": "agent.router",
  "to": "human.adam",
  "intent": "demo.hello.world",
  "corr": "01EXAMPLE_CORR_0000000000001",
  "reply_to": "01EXAMPLE_TASK_0000000000001",
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {"ok": true, "note": "Same corr. reply_to points to the task."},
  "sig": null
}
```

> `human.adam → agent.router → agent.worker → result` — same `corr`, `reply_to` bubbles back

> **AEE exists because agents hate ambiguity** — fixed envelope, computable causality, fewer tokens.

> If an agent can parse it, a human can grep it.

---

## What an Agent Sees

**Without AEE** — ambiguous, unrecoverable:

```
{"task": "check backup", "from": "someone", "data": {...}}
```

What's missing:
- **No `corr`** → Cannot link to workflow. If this fails, no retry context.
- **No `reply_to`** → Cannot trace what triggered this. Causality is lost.
- **No `intent`** → Must infer meaning from payload. Fragile.
- **No `to`** → Broadcast? Direct? Unknown.

**With AEE** — structured, recoverable:

```
{"from": "agent.router", "to": "agent.worker", "corr": "xyz-789", "reply_to": "bbb-222", "intent": "backup.check", ...}
```

Every field answers a question. No inference required.

---

## What an LLM Actually Processes

**Free-form prompt** — variable structure, implicit context:

```
"Please check the backup status. This is from the router.
Reply when done. The original request was from Adam."
```

The LLM must:
- Parse natural language to extract intent
- Infer who to reply to
- Guess at correlation
- Hope the context survives

**AEE envelope** — fixed structure, explicit context:

```json
{"from": "agent.router", "to": "agent.worker", "intent": "backup.check", "corr": "xyz-789", "reply_to": "bbb-222"}
```

The LLM (or any parser):
- Reads 5 predictable keys
- Zero inference, zero ambiguity
- Fewer tokens, faster parsing
- Correlation is computable, not guessed

**Result:** Lower cost, higher reliability, deterministic traceability.

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

**→ [Protocol Handshake](examples/handshake.md)** — How agents discover supported intents (discovery, not orchestration).

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

- **[MVE (Minimal Viable Envelope)](aee.md#10-adoption-tiers)** — Start with 5 fields for logging, or 10 for full compliance
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

See [GitHub Issues](https://github.com/quoxai/aee/issues) for planned work:

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

## Related Protocols

AEE is part of the **Quox protocol family** — three complementary specs that together provide messaging, control, and evidence for agentic systems:

| Protocol | Role | Repo |
|----------|------|------|
| **AEE** | Envelope format + causality | *(this repo)* |
| **AOCL** | Orchestration control layers (policy, routing, HITL gates) | [AOCL](https://github.com/quoxai/aocl) |
| **VOLT** | Verifiable evidence ledger + tamper-evident traces | [VOLT](https://github.com/quoxai/volt) |

**How they connect:**
- AEE envelopes carry `corr` and `reply_to` — AOCL uses these for layer-level audit, VOLT uses them as `correlation_id` for evidence chains.
- AOCL emits `aocl.*` intents as AEE envelopes — no AEE spec changes needed.
- VOLT records AEE envelope events (`aee.envelope.received`, `aee.envelope.sent`) as part of its tamper-evident trace.

Each protocol is independently useful. Together they provide **observable, controllable, provable** agent operations.

## License

See [LICENSE](LICENSE) for details.
