# AEE v1 — Agent Envelope Exchange (Single-Page Spec)

**Status:** Draft  
**Goal:** A tiny, canonical **message envelope** for agent↔agent comms that is:
- portable across frameworks (LangGraph/AutoGen/etc.)
- machine-validated (JSON Schema)
- human-debuggable (readable logs)
- observable (trace/correlation)
- extensible without bloat (intent schemas live outside the envelope)

AEE standardizes the **envelope** only. The meaning lives in the `intent` namespace + the `payload` schema for that intent.

---

## 1) The Envelope (14 fields)

An AEE message is a JSON object with the following top-level fields:

1. `v` *(string)* — envelope schema version (e.g. `"1"`)
2. `id` *(string)* — unique message ID (ULID/UUID recommended)
3. `ts` *(string)* — timestamp (ISO 8601 UTC recommended)
4. `type` *(string)* — one of: `task | result | event | error | stream`
5. `from` *(string)* — sender agent ID (e.g. `"agent.monitor.pve"`)
6. `to` *(string)* — recipient agent ID or channel (e.g. `"agent.manager"` or `"bus.ops"`)
7. `intent` *(string)* — namespaced verb+noun (e.g. `"ops.backup.status.check"`)
8. `corr` *(string)* — correlation ID for a whole workflow/thread
9. `reply_to` *(string|null)* — `id` of message being replied to (MUST be non-null for `result`/`error`, null otherwise)
10. `trace` *(object|null)* — tracing hooks `{trace_id, span_id}` (optional but recommended)
11. `priority` *(string)* — `low | normal | high | urgent`
12. `requires` *(object|null)* — constraints/expectations (timeouts, approvals, evidence)
13. `payload` *(object)* — intent-specific data (validated by intent schema)
14. `sig` *(object|string|null)* — optional signature/auth proof

**Principle:** Keep the envelope stable and small. Everything else is payload schema or optional extensions.

---

## 2) Field Semantics

### 2.1 `type`
- `task`: request to do work (expects `result` or `error`)
- `result`: successful completion of a `task`
- `event`: informational signal (no reply required)
- `error`: unsuccessful completion of a `task`
- `stream`: partial/progress update related to a `task` or long-running process

### 2.2 `intent`
A stable, namespaced identifier:
- format: flexible dotted notation (recommended: `domain.subdomain.noun.verb`)
- examples:
  - `ops.network.port.probe`
  - `ops.backup.status.check`
  - `infra.proxmox.vm.create`
  - `docs.summarize.with_citations`

**Rule:** If two agents can't agree on `intent`, they don't share semantics.

> **Note:** AEE does not prescribe transport; envelopes may be carried over any reliable or unreliable channel.

### 2.3 `requires`
A small "contract" for execution. Common keys:
- `timeout_ms` *(number)*
- `human_approval` *(boolean)*
- `evidence` *(boolean)* — whether proof/artifacts must be returned
- `format` *(string)* — e.g. `"table"`, `"json"`, `"markdown"`
- `min_confidence` *(number 0..1)*

Agents may ignore unknown keys. Producers should keep this compact.

**Example:**
```json
{"timeout_ms": 30000, "evidence": true, "human_approval": false}
```

### 2.4 `sig`
Optional. If used, it should bind at least:
- `v,id,ts,type,from,to,intent,corr,reply_to,payload`

Example `sig` objects:
- `{ "alg": "HS256", "kid": "key-1", "value": "<jwt-or-hmac>" }`
- `{ "alg": "ed25519", "kid": "key-2", "value": "<signature>" }`

---

## 3) Validity Rules (Normative)

1. `v,id,ts,type,from,to,intent,corr,priority,payload` **MUST** exist.
2. `type` **MUST** be one of: `task|result|event|error|stream`.
3. If `type` is `result` or `error`, `reply_to` **MUST** be non-null.
4. `payload` **MUST** be an object (intent schema defines its shape).
5. Unknown fields **MUST** be ignored (forward compatibility).
6. Consumers **SHOULD** log `id`, `corr`, `intent`, `from`, `to`, `type`, `priority`.
7. Producers **SHOULD** include `trace` if an observability system exists.

---

## 4) Minimal Examples

### 4.1 Task

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
  "requires": {"timeout_ms": 30000, "evidence": true, "human_approval": false},
  "payload": {"cluster": "node.lan", "window": "24h"},
  "sig": null
}
```

### 4.2 Result

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

### 4.3 Error

```json
{
  "v": "1",
  "id": "01JFB2T3W9H0J1FQWQ7B2M3D5E",
  "ts": "2025-12-14T03:45:21Z",
  "type": "error",
  "from": "agent.backup_auditor",
  "to": "agent.manager",
  "intent": "ops.backup.status.check",
  "corr": "01JFB2QX0K8X5K6ZJ9G2C0C1MW",
  "reply_to": "01JFB2R1JZKQ9V3K8W8Y9W1F2A",
  "trace": {"trace_id": "9f3c", "span_id": "c01d"},
  "priority": "high",
  "requires": {"evidence": true},
  "payload": {
    "code": "E_TIMEOUT",
    "message": "PBS API did not respond within 30s",
    "retryable": true,
    "backoff_ms": 60000,
    "evidence_refs": ["tcp:pve02->pbs01:8007"]
  },
  "sig": null
}
```

---

## 5) Intent Schemas (Where “Shared Semantics” Lives)

AEE does **not** standardize `payload`. Instead, each `intent` publishes:

* `task.payload` schema
* `result.payload` schema
* `error.payload` schema (optional but recommended)

**Recommended layout in a repo:**

* `envelope/aee-v1.schema.json`
* `intents/ops.backup.status.check.task.schema.json`
* `intents/ops.backup.status.check.result.schema.json`
* `intents/ops.backup.status.check.error.schema.json`

**Why this is enough:**
The envelope lets messages route, correlate, trace, and enforce basic contracts. The intent schemas supply the domain meaning. Together they form “structured intent + shared semantics”.

---

## 6) JSON Schema (Envelope) — aee-v1.schema.json

> This schema validates the envelope shape only (not intent-specific payloads).

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.org/aee/aee-v1.schema.json",
  "title": "AEE v1 Envelope",
  "type": "object",
  "additionalProperties": true,
  "required": ["v","id","ts","type","from","to","intent","corr","priority","payload"],
  "properties": {
    "v": {"type": "string", "const": "1"},
    "id": {"type": "string", "minLength": 8},
    "ts": {"type": "string", "minLength": 10},
    "type": {"type": "string", "enum": ["task","result","event","error","stream"]},
    "from": {"type": "string", "minLength": 1},
    "to": {"type": "string", "minLength": 1},
    "intent": {"type": "string", "minLength": 3},
    "corr": {"type": "string", "minLength": 8},
    "reply_to": {"type": ["string","null"]},
    "trace": {
      "type": ["object","null"],
      "additionalProperties": true,
      "properties": {
        "trace_id": {"type": "string"},
        "span_id": {"type": "string"}
      }
    },
    "priority": {"type": "string", "enum": ["low","normal","high","urgent"]},
    "requires": {"type": ["object","null"], "additionalProperties": true},
    "payload": {"type": "object", "additionalProperties": true},
    "sig": {"type": ["object","string","null"], "additionalProperties": true}
  },
  "allOf": [
    {
      "if": {"properties": {"type": {"enum": ["result","error"]}}},
      "then": {"required": ["reply_to"], "properties": {"reply_to": {"type": "string", "minLength": 8}}}
    }
  ]
}
```

---

## 7) Conformance Tests (Recommended)

A conforming implementation should:

* accept the sample messages above as valid
* reject messages missing required fields
* enforce `reply_to` for `result` and `error`
* preserve unknown fields (do not strip by default)

---

## 8) Roadmap (Non-Bloat Extensions)

Keep AEE v1 stable. Add extensions as separate documents:

* **Intent Registry Conventions:** naming, versioning, discovery
* **Error Codes Catalog:** optional standard codes
* **Security Profile:** signing + key distribution + replay protection
* **Transport Bindings:** HTTP, WebSocket, NATS, Kafka, Redis streams, etc.

---

## 9) Why AEE Exists (One Sentence)

**AEE makes agent messages portable and composable by standardizing the envelope while letting communities standardize meaning through intent-specific schemas.**
