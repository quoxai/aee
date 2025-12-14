# AEE Intent Registry — Starter Set

This document defines a **small, illustrative set of intent schemas** for use with **AEE v1 (Agent Envelope Exchange)**.

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
