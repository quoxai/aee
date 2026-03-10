# AEE Intent Registry — Starter Set

This document defines a **small, illustrative set of intent schemas** for use with **AEE v1 (Agent Envelope Exchange)**.

> **Note:** The `aee.*` namespace is reserved for protocol negotiation. Application intents should use other namespaces.

---

## Reserved Protocol Intents (`aee.*`)

The `aee.*` namespace is **reserved for protocol negotiation and context retrieval only**. These intents enable agents to discover capabilities, fetch context, and validate payloads without custom integration.

> If you find yourself adding scheduling, workflow, or orchestration semantics to `aee.*`, you're leaving AEE and building an orchestrator.

---

### aee.status.ping

Liveness check. Responds with pong.

#### Task Payload Schema

```json
{
  "type": "object",
  "properties": {},
  "additionalProperties": false
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["pong"],
  "properties": {
    "pong": {"type": "boolean", "const": true},
    "ts": {"type": "string", "description": "Response timestamp (ISO 8601)"}
  }
}
```

---

### aee.status.health

Health and readiness status.

#### Task Payload Schema

```json
{
  "type": "object",
  "properties": {},
  "additionalProperties": false
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["status"],
  "properties": {
    "status": {"type": "string", "enum": ["healthy", "degraded", "unhealthy"]},
    "details": {"type": "object", "additionalProperties": true}
  }
}
```

---

### aee.spec.query

Return supported AEE version and capabilities.

#### Task Payload Schema

```json
{
  "type": "object",
  "properties": {},
  "additionalProperties": false
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["aee_version"],
  "properties": {
    "aee_version": {"type": "string", "description": "Supported AEE envelope version"},
    "extensions": {"type": "array", "items": {"type": "string"}, "description": "Optional extensions supported"}
  }
}
```

---

### aee.capability.list

List supported intents.

#### Task Payload Schema

```json
{
  "type": "object",
  "properties": {
    "namespace_filter": {"type": "string", "description": "Optional: filter by intent namespace prefix"}
  }
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["intents"],
  "properties": {
    "intents": {
      "type": "array",
      "items": {"type": "string"},
      "description": "List of supported intent identifiers"
    }
  }
}
```

---

### aee.context.fetch

Retrieve envelope or payload by reference.

#### Task Payload Schema

```json
{
  "type": "object",
  "required": ["id"],
  "properties": {
    "id": {"type": "string", "description": "Envelope ID to retrieve"},
    "corr": {"type": "string", "description": "Optional: correlation ID for context"},
    "locator": {"type": "string", "description": "Optional: URI or path hint"},
    "hash": {"type": "string", "description": "Optional: expected content hash for verification"}
  }
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["found"],
  "properties": {
    "found": {"type": "boolean"},
    "envelope": {"type": "object", "description": "The retrieved envelope (if found)"},
    "hash": {"type": "string", "description": "Content hash of returned envelope"}
  }
}
```

---

### aee.context.refute

Reject a referenced context with reason.

#### Task Payload Schema

```json
{
  "type": "object",
  "required": ["id", "reason"],
  "properties": {
    "id": {"type": "string", "description": "Envelope ID being refuted"},
    "reason": {"type": "string", "description": "Why this context is being rejected"}
  }
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["acknowledged"],
  "properties": {
    "acknowledged": {"type": "boolean"}
  }
}
```

---

### aee.validate.payload

Validate a payload against an intent schema.

#### Task Payload Schema

```json
{
  "type": "object",
  "required": ["intent", "payload_to_validate"],
  "properties": {
    "intent": {"type": "string", "description": "Intent whose schema should be used"},
    "payload_to_validate": {"type": "object", "description": "The payload to validate"},
    "type_hint": {"type": "string", "enum": ["task", "result", "error"], "description": "Which schema variant to validate against"}
  }
}
```

#### Result Payload Schema

```json
{
  "type": "object",
  "required": ["valid"],
  "properties": {
    "valid": {"type": "boolean"},
    "errors": {
      "type": "array",
      "items": {"type": "string"},
      "description": "Validation error messages (if invalid)"
    }
  }
}
```

---

---

## Extension Namespace (`aee.ext.*`)

The `aee.ext.*` namespace is for **optional protocol extensions** that enhance AEE capabilities without modifying the core envelope. Extensions are documented here for interoperability but are not required for AEE compliance.

### aee.ext.decision_evidence

Structured decision evidence capture. When `requires.decision_evidence` is set on a `task` envelope, the responding agent includes a `decision_evidence` object in the `result` payload.

See [AEE spec Section 13](aee.md#13-decision-evidence) for the full specification.

**Schema:** [`schemas/decision-evidence.schema.json`](schemas/decision-evidence.schema.json)

**Levels:** `"none"` | `"minimal"` | `"standard"` | `"full"`

**Usage in `requires`:**
```json
{"decision_evidence": "standard"}
```

**Result payload includes:**
```json
{
  "payload": {
    "decision_evidence": {
      "inputs_used": ["fleet health data", "last backup log"],
      "tools_used": ["ssh.exec"],
      "decision": "Escalated to human operator",
      "reason_summary": "Backup failure on 2 of 3 nodes exceeds auto-recovery threshold",
      "action_taken": "Created approval request for manual intervention",
      "confidence": 0.88
    }
  }
}
```

**Integration:** Decision evidence objects are captured by VOLT (tamper-evident chain) and witnessed by WARD (content-free receipts), creating a three-layer trust chain.

---

## Application Intents

These intents are:
- broadly useful across domains
- simple enough to understand at a glance
- sufficient to demonstrate "structured intent + shared semantics"

AEE does **not** require a centralized registry. This file exists as:
- a reference
- a starting vocabulary
- an example of how communities may publish intent schemas

---

## Intent Naming Conventions

Recommended (not required):

```
<domain>.<subdomain>.<noun>.<verb>
```

Examples:
- `ops.network.port.probe`
- `ops.backup.status.check`
- `docs.summarize.with_citations`

Rules of thumb:
- verbs should be explicit (`check`, `probe`, `create`, `summarize`)
- avoid overloading intents with optional behavior
- version intent schemas independently if needed

---

## 1) ops.network.port.probe

Probe network reachability for a TCP port.

### Task Payload Schema

```json
{
  "type": "object",
  "required": ["source", "target", "port"],
  "properties": {
    "source": {"type": "string", "description": "origin node or agent"},
    "target": {"type": "string", "description": "hostname or IP"},
    "port": {"type": "integer", "minimum": 1, "maximum": 65535},
    "timeout_ms": {"type": "integer"}
  }
}
```

### Result Payload Schema

```json
{
  "type": "object",
  "required": ["reachable"],
  "properties": {
    "reachable": {"type": "boolean"},
    "latency_ms": {"type": "number"},
    "confidence": {"type": "number", "minimum": 0, "maximum": 1},
    "evidence_refs": {"type": "array", "items": {"type": "string"}}
  }
}
```

### Error Payload Schema

```json
{
  "type": "object",
  "required": ["code", "message"],
  "properties": {
    "code": {"type": "string"},
    "message": {"type": "string"},
    "retryable": {"type": "boolean"},
    "evidence_refs": {"type": "array", "items": {"type": "string"}}
  }
}
```

---

## 2) ops.backup.status.check

Check the status of recent backup jobs.

### Task Payload Schema

```json
{
  "type": "object",
  "required": ["cluster", "window"],
  "properties": {
    "cluster": {"type": "string"},
    "window": {"type": "string", "description": "time window, e.g. '24h', 'last_7d'"}
  }
}
```

### Result Payload Schema

```json
{
  "type": "object",
  "required": ["status"],
  "properties": {
    "status": {
      "type": "string",
      "enum": ["OK", "PARTIAL_FAILURE", "FAILURE"]
    },
    "failed": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["node", "reason"],
        "properties": {
          "node": {"type": "string"},
          "reason": {"type": "string"}
        }
      }
    },
    "confidence": {"type": "number", "minimum": 0, "maximum": 1},
    "evidence_refs": {"type": "array", "items": {"type": "string"}}
  }
}
```

### Error Payload Schema

```json
{
  "type": "object",
  "required": ["code", "message"],
  "properties": {
    "code": {"type": "string"},
    "message": {"type": "string"},
    "retryable": {"type": "boolean"},
    "backoff_ms": {"type": "integer"},
    "evidence_refs": {"type": "array", "items": {"type": "string"}}
  }
}
```

---

## 3) docs.summarize.with_citations

Summarize a document while preserving traceable sources.

### Task Payload Schema

```json
{
  "type": "object",
  "required": ["input"],
  "properties": {
    "input": {"type": "string", "description": "raw text or reference"},
    "max_length": {"type": "integer"},
    "style": {"type": "string"}
  }
}
```

### Result Payload Schema

```json
{
  "type": "object",
  "required": ["summary", "citations"],
  "properties": {
    "summary": {"type": "string"},
    "citations": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["ref", "excerpt"],
        "properties": {
          "ref": {"type": "string"},
          "excerpt": {"type": "string"}
        }
      }
    },
    "confidence": {"type": "number", "minimum": 0, "maximum": 1}
  }
}
```

### Error Payload Schema

```json
{
  "type": "object",
  "required": ["code", "message"],
  "properties": {
    "code": {"type": "string"},
    "message": {"type": "string"},
    "retryable": {"type": "boolean"}
  }
}
```

---

## Notes on Evolution

* New intents can be published independently
* Intent schemas may version without changing AEE
* Agents may advertise supported intents via a separate capability mechanism
* Absence of an intent means absence of shared semantics — not an error

---

## Why This Matters

AEE standardizes **how agents speak**.  
Intent schemas standardize **what they mean**.

Together, they enable interoperable, auditable, agentic workflows without forcing a single framework, runtime, or vendor.
