# Engineering Standard (Spawn/Mesh)

## Scope
Applies to daemon/services and supporting adapters in `spawn` and `mesh`.

## 1) Architecture Rules
- Use `functional core, imperative shell`.
- Keep domain/state logic pure and testable.
- Keep systemd/process/network/file IO in adapter modules.
- No direct privileged operations from core logic.
- Target split:
  - Rust for daemon core + critical adapters.
  - Python for thin orchestration and operator UX.

## 2) Contracts and Versioning
- Define versioned schemas for:
  - config
  - event envelope
  - action request/result
  - state transitions
- Compatibility policy:
  - additive fields only for minor releases
  - breaking changes require explicit major bump and migration notes
- Codegen policy:
  - generate API clients/stubs from OpenAPI/proto
  - keep operator CLI hand-authored as a thin layer over generated clients

## 3) systemd Standards
- Prefer `Type=notify` with readiness signaling.
- Use watchdog heartbeats when long-running.
- Set explicit:
  - `Restart=` policy
  - `TimeoutStartSec` / `TimeoutStopSec`
  - resource limits and hardening flags
- Run unprivileged by default.

## 4) Observability
- Emit structured JSON logs.
- Required log fields:
  - `ts`, `level`, `component`, `event`, `request_id`, `result`
- Propagate correlation IDs end-to-end.
- Provide health/readiness checks.

## 5) Safety and Reliability
- All actions must be idempotent.
- Use idempotency keys on retriable flows.
- Fail fast on invalid config.
- No silent fallback on critical settings.
- Keep rollback metadata for reversible actions.

## 6) Testing Policy
- Unit tests for core state logic.
- Property tests (Hypothesis) for parsers, normalizers, transitions.
- Integration tests for adapter boundaries.
- CI gates:
  - lint/format
  - schema validation
  - test suite

## 7) Security
- Least privilege for services and helper binaries.
- Validate/sanitize all external inputs.
- Allowlist command/tool execution.
- Keep audit trail: event -> decision -> action -> outcome.

## 8) Migration and Compatibility
- Compatibility shims allowed only with explicit deprecation timeline.
- Emit deprecation warnings in logs.
- Publish migration steps in release notes/docs.

## 9) PR Checklist
- Contracts updated and validated.
- Tests added/updated (unit + property/integration as relevant).
- Logging includes required fields.
- systemd unit changes reviewed for hardening and restart behavior.
- Docs updated for operator impact.
