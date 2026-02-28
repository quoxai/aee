# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Repository housekeeping and GitHub templates
- MVE (Minimal Viable Envelope) adoption tiers: MVE-Required (schema-valid) and MVE-5 (log-friendly)
- Reserved `aee.*` protocol intent namespace with 7 seed intents (ping, health, spec.query, capability.list, context.fetch, context.refute, validate.payload)
- Human↔Agent symmetry: clarified `from`/`to` fields support `human.*` identifiers as first-class entities
- Wrap-by-reference guidance: recommended `payload.references` shape to avoid nested envelope bloat
- Middleware adoption strategy documentation for gateway-layer AEE injection

## [0.1.0] - 2025-12-14

### Added
- Initial AEE specification and documentation
- Core envelope structure definition (14 fields)
- Intent-based messaging framework
- Starter intent registry with example schemas
- JSON Schema for envelope validation
- Documentation on relationship to MCP and ACP
- Example task/result/error message envelopes

[0.1.0]: https://github.com/quoxai/aee/releases/tag/v0.1.0
