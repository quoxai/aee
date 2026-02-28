# Protocol Handshake: Capability Discovery

> **This is discovery, not orchestration.** AEE does not schedule, route, or run workflows. These intents let agents ask "what do you support?" — nothing more.

---

## `aee.capability.list` — Ask What Intents Are Supported

**Task:** Agent A asks Agent B what intents it handles.

```json
{
  "v": "1",
  "id": "01HANDSHAKE_TASK_00000000001",
  "ts": "2025-12-23T15:00:00Z",
  "type": "task",
  "from": "agent.coordinator",
  "to": "agent.worker",
  "intent": "aee.capability.list",
  "corr": "01HANDSHAKE_CORR_00000000001",
  "reply_to": null,
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {},
  "sig": null
}
```

**Result:** Agent B responds with its supported intents.

```json
{
  "v": "1",
  "id": "01HANDSHAKE_RESULT_0000000001",
  "ts": "2025-12-23T15:00:01Z",
  "type": "result",
  "from": "agent.worker",
  "to": "agent.coordinator",
  "intent": "aee.capability.list",
  "corr": "01HANDSHAKE_CORR_00000000001",
  "reply_to": "01HANDSHAKE_TASK_00000000001",
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {
    "intents": [
      "ops.backup.check",
      "ops.backup.run",
      "ops.network.probe",
      "aee.capability.list",
      "aee.status.ping"
    ]
  },
  "sig": null
}
```

**What just happened:**
- Agent A sent a `task` with `intent: aee.capability.list`
- Agent B replied with a `result` listing its supported intents
- Same `corr` links both envelopes
- `reply_to` points from result to task

---

## `aee.spec.query` — Ask for AEE Version Info

**Task:** Check what AEE version an agent supports.

```json
{
  "v": "1",
  "id": "01SPECQUERY_TASK_00000000001",
  "ts": "2025-12-23T15:01:00Z",
  "type": "task",
  "from": "agent.coordinator",
  "to": "agent.worker",
  "intent": "aee.spec.query",
  "corr": "01SPECQUERY_CORR_00000000001",
  "reply_to": null,
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {},
  "sig": null
}
```

**Result:**

```json
{
  "v": "1",
  "id": "01SPECQUERY_RESULT_0000000001",
  "ts": "2025-12-23T15:01:01Z",
  "type": "result",
  "from": "agent.worker",
  "to": "agent.coordinator",
  "intent": "aee.spec.query",
  "corr": "01SPECQUERY_CORR_00000000001",
  "reply_to": "01SPECQUERY_TASK_00000000001",
  "trace": null,
  "priority": "normal",
  "requires": null,
  "payload": {
    "aee_version": "1",
    "extensions": []
  },
  "sig": null
}
```

---

## When to Use These

| Intent | Use Case |
|--------|----------|
| `aee.capability.list` | Before delegating work, check if the target agent supports the intent you need |
| `aee.spec.query` | Verify AEE version compatibility before sending complex envelopes |
| `aee.status.ping` | Health check (not shown here — see [intents.md](../intents.md)) |

---

## What This Is NOT

- **Not routing** — AEE doesn't decide where to send messages
- **Not scheduling** — AEE doesn't queue or retry
- **Not orchestration** — AEE doesn't coordinate multi-step workflows

These intents are **protocol negotiation**: agents asking each other "what do you speak?" before speaking.
