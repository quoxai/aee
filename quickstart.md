# Quickstart: Your First AEE Envelope

**You do not need to understand AEE yet.**

Just copy, paste, and emit. Understanding comes later.

---

## 1. The Envelope (Copy This)

```json
{
  "v": "1",
  "id": "01EXAMPLE00000000000000001",
  "ts": "2025-12-23T12:00:00Z",
  "type": "task",
  "from": "human.you",
  "to": "agent.something",
  "intent": "demo.hello.world",
  "corr": "01EXAMPLE00000000000000001",
  "reply_to": null,
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {
    "message": "Hello from a human."
  },
  "sig": null
}
```

That's a valid AEE envelope. You just made one.

---

## 2. Emit It (Python, 10 Lines)

```python
import json
import uuid
from datetime import datetime, timezone

envelope = {
    "v": "1",
    "id": str(uuid.uuid4()),
    "ts": datetime.now(timezone.utc).isoformat(),
    "type": "task",
    "from": "human.you",
    "to": "agent.something",
    "intent": "demo.hello.world",
    "corr": str(uuid.uuid4()),
    "reply_to": None,
    "trace": None,
    "priority": "normal",
    "requires": None,
    "payload": {"message": "Hello from a human."},
    "sig": None
}

print(json.dumps(envelope, indent=2))
```

Run it. You'll see a valid AEE envelope in your terminal.

---

## 3. What You Just Did

- `from`: You said who you are (`human.you`)
- `to`: You said who you're talking to (`agent.something`)
- `intent`: You said what you want (`demo.hello.world`)
- `corr`: You gave this workflow an ID that survives every hop
- `payload`: You put your actual data here

Everything else is either optional or protocol scaffolding.

---

## 4. What Happens Next

When `agent.something` replies, it sends back an envelope with:
- `type: "result"` (or `"error"`)
- `reply_to`: your envelope's `id`
- `corr`: same as yours

Now you can trace the whole conversation.

---

## 5. The Causality Chain

Here's what traceability looks like across multiple hops:

```
human.you
    ↓ id: aaa-111, corr: xyz-789
agent.router
    ↓ id: bbb-222, corr: xyz-789, reply_to: aaa-111
agent.worker
    ↓ id: ccc-333, corr: xyz-789, reply_to: bbb-222
result
    ↓ id: ddd-444, corr: xyz-789, reply_to: ccc-333
```

Every message shares the same `corr`. Every response points to what it's replying to. The chain is traceable from human to final result.

---

## 6. Want to Learn More?

- **[Full Spec (aee.md)](aee.md)** — All 14 fields explained
- **[Intent Registry (intents.md)](intents.md)** — Example intents with schemas
- **[Adoption Tiers (aee.md#adoption-tiers)](aee.md#adoption-tiers)** — Start small with MVE-5

**But you don't have to read those yet.** You already emitted a valid envelope. That's the point.
