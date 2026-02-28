# Fabric Operation Model

## Purpose
Unify `sysops`, `devops`, and `secops` under a single control fabric where `systemd` is the controller, `watcher` is the event router, and operator+AI collaboration is policy-gated and auditable.

## Naming Conventions
- Codex fabric model: `spawn` (previously `dome`)
- Codex daemon: `spawnd` (preferred; `domed` retained as compatibility name)
- Ops router entrypoint: `meshctl` (with `watchctl` compatibility alias)

## Runtime Direction
- End-state architecture:
  - Rust: core runtime + system/privileged adapters (`spawnd`, `meshd`)
  - Python: thin clients + orchestration workflows (`spawnctl`, `meshctl`)
- Contract boundary remains spec-first and language-neutral.

## Core Loop
1. Sense: collect host/network/tooling signals.
2. Normalize: convert signals to typed events.
3. Decide: evaluate policy/state transitions.
4. Act: execute bounded actions via systemd/services/scripts.
5. Verify: capture outcomes and evidence.
6. Tune: update thresholds/rules using feedback.

## Planes
- Control Plane: specs, policies, transition rules, unit dependency graph.
- Data Plane: telemetry/baselines/events/evidence artifacts.
- Execution Plane: action handlers and service orchestration.
- Experience Plane: operator surfaces (dashboards, shells, layouts, docs).

## Controllers and Roles
- `systemd`:
  - lifecycle, ordering, retries, timers, targets
  - source of truth for service state
- `watcher`:
  - route events to profiles/actions
  - enforce command allowlists and debounce/cooldowns
  - emit execution logs/events
  - preferred control binary: `meshctl` (with `watchctl` compatibility alias)
- `AI agent`:
  - triage, enrichment, pattern correlation, proposal generation
  - no implicit high-risk execution
- `Operator`:
  - approves risk-bearing transitions/actions
  - owns policy and final authority

## State and Transition Model
- Known good: `BASELINE_OK`, `EXPECTED_DRIFT`
- Known suspicious: `ANOMALY_OBSERVED`, `ENRICHING`
- Known bad: `IOC_CONFIRMED`, `POLICY_VIOLATION`, `INCIDENT_OPEN`

Transition rules are explicit, versioned, and testable. High-impact transitions require operator gate.

## Action Classes
- `observe`: log/collect only
- `warn`: notify + enrich
- `enforce_safe`: reversible bounded actions
- `contain`: medium/high gate (network/workflow restrictions)
- `promote`: incident artifact + escalation

All actions must include:
- policy/rule version
- preconditions
- timeout
- rollback metadata (when reversible)
- evidence references

## Guardrails
- Typed inputs/outputs (schema-validated)
- Idempotency keys for retried events/actions
- Allowlisted commands/tools only
- Default-deny for destructive operations
- Full audit trail (event -> decision -> action -> outcome)

## Collaboration Contract (Operator + AI)
- AI may propose and execute only policy-permitted low-risk steps.
- AI must attach evidence and confidence to decisions.
- Operator approval required for containment/hard actions.
- Post-action feedback labels (`tp`, `fp`, `needs_tuning`) feed policy tuning.

## Portability
This model is implemented first in `watcher`, but is fabric-level and reusable across repos/services.
