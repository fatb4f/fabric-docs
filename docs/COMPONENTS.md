# Components

## Overview
This document maps the platform fabric into concrete components and interfaces.

## Category Split
### Codex-specific
- `watcher/bin/codex-event-source`
- `watcher/bin/codex-config-validate`
- `watcher/bin/codex-resume-diagnose`
- `watcher/bin/codex-alert-log`
- codex session/config state under `~/.config/codex` and `~/.local/state/codex`
- profiles and routes tied to Codex session refresh/diagnostics

### Ops-specific
- `systemd` units/timers/targets
- `watcher/bin/watchctl` and non-codex event sources
- `watcher/bin/system-security-package-event-source`
- `vps_secops` triage/guardrail scripts and contracts
- osquery baseline collection and health reports
- infra/runtime wiring from `dotfiles` (zellij/systemd/env)

## Domain Layers by Repo
- `opsctrl`: collaboration UX and paired-dev operations surface.
- `two`: sysops ontology/model vocabulary.
- `oracle`: learning UX for Python-dev workflows.
- `firefox-profile-configs`: webops profile/config generation and policy wiring.

## Control Components
- `systemd` (user/system):
  - service lifecycle
  - dependency ordering
  - timers/targets
  - restart policy and status source of truth
- `watchctl` (`watcher/bin/watchctl`):
  - event routing by `topic`/`topic_prefix`
  - profile dispatch
  - execution logging and failure handling
- `meshctl` (`watcher/bin/meshctl`):
  - preferred operator entrypoint (delegates to `watchctl`)
- Service direction:
  - `spawnd` and `meshd` become primary daemon identities
  - `spawnctl` and `meshctl` are thin operator/orchestration clients

## Data Components
- Event sources:
  - `watcher/bin/codex-event-source`
  - `watcher/bin/system-security-package-event-source`
  - `vps_secops/scripts/vps_watcher_event_source.py`
- Baseline/telemetry:
  - osquery packs and health outputs
  - journald/pacman-derived events
- Artifact stores:
  - JSONL logs under `~/.local/state/...`
  - pack outputs and incident artifacts

## Execution Components
- Profile commands (`watcher/config/profiles*.yaml`)
- Guardrail/action scripts (`vps_secops/scripts/*`)
- Host control primitives:
  - `systemctl --user` and system units
  - network policy handlers (sinkhole/lockdown patterns)

## Contract Components
- JSON Schemas: event/decision/action contracts
- OpenAPI: operator/control-plane API
- Proto/gRPC: typed internal actions and service boundaries
- Generation/check scripts:
  - `scripts/gen.sh`
  - `scripts/check_gen.sh`
- Language split contract:
  - Rust services implement core/adapters behind the same contracts
  - Python clients are generated + thin orchestration wrappers

## Collaboration Components
- Operator:
  - policy ownership
  - high-gate approvals
  - incident adjudication
- AI agent:
  - triage, enrichment, correlation, proposals
  - constrained execution under policy

## UX Components
- Zellij ops layouts (`dotfiles/dot_config/zellij/layouts/*`)
- Tails/dashboards:
  - `watchctl-tail-events`
  - `watchctl-tail-alerts`
  - `watchctl-control-dashboard`
  - `watchctl-systemd-status-pane`

## Core Interfaces
- Event envelope:
  - required: `topic`, `source`, `ts`
  - optional: domain payload fields
- Decision output:
  - `decision`, `confidence`, `evidence_refs`, `rule_id`, `policy_version`
- Action output:
  - `action_type`, `status`, `result`, `evidence_refs`, `rollback_ref`

## Gate Levels
- `low`: observe/enrich/report (automatic)
- `medium`: reversible enforcement (policy threshold)
- `high`: containment/risk-bearing actions (operator approval)

## Failure Domains
- Source failure: event stops at producer -> alert and source restart policy.
- Router failure: watchctl dispatch failure -> logged + optional codex diagnosis.
- Action failure: non-zero command -> evidence + retry/backoff policy.
- Contract drift: generation/check failure -> block merge/deploy.

## Minimal Runtime Path
1. Source emits JSONL event.
2. `watchctl daemon` matches route.
3. Profile command executes.
4. Output logged as structured JSON.
5. Feedback updates policy tuning inputs.
