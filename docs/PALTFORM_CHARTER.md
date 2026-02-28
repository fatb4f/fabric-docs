# Platform Charter

## Mission
Operate local and distributed infrastructure through a policy-driven, systemd-centered control fabric where AI accelerates triage and execution while operators retain authority over risk-bearing actions.

## Non-Negotiables
- Spec-first contracts for events, decisions, and actions.
- Deterministic orchestration via systemd units/timers/targets.
- Explicit policy gates for risky transitions.
- Complete auditability for every automated action.
- Human override is always available.

## Scope
- Integrate `sysops`, `devops`, `secops` workflows.
- Standardize event ingestion, normalization, routing, and response.
- Maintain an operator-friendly local control surface.

## Repo Roles Under `~/src`
- `watcher`: event routing/runtime execution engine and dashboard helpers.
- `codex-dome`: codex fabric model (`spawn`) and daemon runtime (`spawnd`).
- `vps_secops`: secops contracts, triage/guardrail model, DFIR scaffolding.
- `spec-hydra`: spec/tooling pattern reference (contracts, generation, tests).
- `dotfiles`: host UX/runtime wiring (systemd user units, shells, launchers, layouts).
- `opsctrl`: UX layer for collaboration and paired development workflows.
- `two`: sysops ontology and model vocabulary layer.
- `oracle`: UX layer for learning Python development and guided experimentation.
- `firefox-profile-configs`: webops profile generation/linking and browser policy artifacts.
- `codex-tunnel`: remote-access/transport bridge utilities.
- `identity-graph`: identity/domain mapping assets and transforms.

## Operating Principles
1. Events before actions.
2. Contracts before implementations.
3. Safe automation before autonomous containment.
4. Feedback-driven tuning over static rule drift.
5. Small composable services over opaque monolith logic.

## Architecture Model
- Controller: `systemd`
- Router: `watchctl`
- Contract layer: JSON Schema + OpenAPI + Proto
- Executors: watcher profiles, scripts, service units
- Feedback: outcome labels + evidence for policy tuning

Target implementation split:
- Rust-first substrate for daemon core and critical adapters.
- Python thin-clients/orchestration for operator workflows and rapid iteration.

## Control Gates
- `low`: observe/enrich/report (auto)
- `medium`: reversible enforcement (policy + confidence gates)
- `high`: containment/destructive potential (operator approval required)

## Definition of Done (Platform Changes)
- Contracts updated and validated.
- Generation/check scripts pass.
- Action path is idempotent and logged.
- Rollback/disable path exists.
- Operator-facing docs updated.

## Out of Scope
- Unbounded autonomous execution.
- Hidden side effects outside audited control paths.
- Policy decisions without versioned artifacts.
