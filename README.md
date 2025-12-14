# AEE — Agent Envelope Exchange

AEE is a tiny, framework-agnostic message envelope for agent-to-agent communication.

It standardizes **how agents exchange messages**, not how they are built, scheduled, or run.

AEE defines a stable envelope (IDs, intent, correlation, tracing) and leaves **meaning** to
intent-specific payload schemas published by communities.

It is designed to be human-debuggable, machine-validated, and transport-agnostic.

AEE complements existing agent frameworks and protocols rather than replacing them.

If agents can agree on an intent, AEE gives them a clean, auditable way to talk.
