# Modern Python CLI + Daemon Stack (for `meshctl`/`meshd` and `spawnctl`/`spawnd`)

## Purpose
Define a practical, OSS-aligned Python stack for CLI + daemon apps with contract-first validation and systemd-native operations.

Positioning:
- Python is the thin-client/orchestration layer.
- Core daemon runtime and critical adapters are expected to converge to Rust.

## 1) CLI Framework
- Primary: `typer`
  - typed args/options, nested commands, shell completion
- Substrate: `click`
  - use directly only for edge/custom parameter behavior

## 2) Terminal UX
- `rich`
  - tables/panels/status for operator-facing output
- `rich.traceback`
  - readable exception rendering for local interactive runs
- `RichHandler`
  - human-readable operator logs in CLI mode

## 3) Contracts, Validation, and Schemas
- `pydantic v2` is canonical for runtime contracts:
  - `event_envelope_v1`
  - `action_request_v1`
  - `action_result_v1`
  - profile/config models
- Export JSON Schema from models with `model_json_schema()`.
- Use `TypeAdapter[...]` for validating plain dict/list payloads.
- Keep schemas versioned and backwards-compatible (additive in minor versions).

## 4) Config Model
- Path substrate:
  - `xdg-base-dirs` for XDG-conformant config/state/cache discovery
  - fallback to explicit env/default paths when unavailable
- File formats:
  - `YAML` via `pyyaml` (operator-facing config)
  - optional `TOML` overlays via `tomllib` (Python 3.11+)
  - optional `JSON` for machine-generated config payloads
- Config loading/composition:
  - `dataconfy` for YAML/JSON loading + environment overlay
  - `pydantic v2` remains the canonical validation layer after load
- Env override strategy:
  - either explicit env mapping in loader
  - or standardized `pydantic-settings` for typed env config
- Rule: fail fast on invalid config; no silent fallback for critical values.

## 5) Logging Model
Use two separate channels:
- Audit logger (machine):
  - JSONL only
  - required fields: `ts`, `component`, `event`, `request_id`, `result`
- Operator logger (human):
  - Rich/console-oriented
  - minimal noise, actionable messages

## 6) Execution Safety
- Use `subprocess` with argv lists (avoid `shell=True` unless explicitly required).
- Capture stdout/stderr and persist to audit artifacts.
- Attach idempotency/correlation IDs to every executed action.

## 7) systemd Integration
- `sdnotify`:
  - `READY=1` on startup complete
  - watchdog pings where enabled
- Unit expectations:
  - explicit restart policy and startup/stop timeouts
  - least privilege and hardening options
- Later extension options:
  - `dbus-next` for richer DBus interactions
  - controlled `systemd-run` wrappers for transient actions

## 8) Testing
- `pytest` baseline
- CLI tests via `click.testing.CliRunner` (Typer-compatible)
- Contract tests:
  - pydantic model validation against golden fixtures
  - normalization/transition invariant tests
- Property tests:
  - `hypothesis` for fuzzing parser/normalizer/transition invariants
- Optional schema snapshot tests for contract drift detection

## 9) Packaging and Distribution
- `pyproject.toml` (PEP 621)
- Console entry points:
  - `meshctl`, `meshd`
  - `spawnctl`, `spawnd`
- Operator install recommendation:
  - `pipx` for isolated per-user CLI installs

## 9.1 CLI Generation Policy (OAPI/Proto)
- Do **not** fully generate operator CLIs from OpenAPI/proto.
- Generate typed clients/stubs from contracts, then build a thin hand-authored CLI on top.
- Generated artifacts should provide:
  - request/response types
  - transport clients
  - validation-ready models
- Hand-authored CLI should own:
  - command UX and subcommand layout
  - policy gates and confirmation prompts
  - multi-step orchestration flows
  - operator-focused output formatting

## 10) Quality Gates
- `ruff` (lint + format)
- `pyright` or `mypy` (recommended for daemon/core modules)
- `pre-commit` hooks for local CI parity
- CI should block on:
  - lint/type/test failures
  - contract/schema validation failures

## 11) Minimal Dependency Set
Runtime:
- `typer`
- `rich`
- `xdg-base-dirs`
- `pydantic>=2`
- `dataconfy`
- `pyyaml`
- `sdnotify`
- `pydantic-settings` (if using settings model)

Dev:
- `pytest`
- `hypothesis`
- `ruff`
- `pyright` or `mypy` (optional but recommended)
- `pre-commit` (recommended)

## 12) Reference Layering
- `core` (pure logic)
  - contracts, state transitions, routing/execution decisions
- `adapters` (imperative shell)
  - CLI (`typer` + `rich`)
  - systemd (`sdnotify`, unit interactions)
  - IO (jsonl, yaml/toml, filesystem, subprocess)

## 13) Recommended Defaults
- Keep CLI and daemon codepaths separate (`*_ctl` vs `*_d` modules).
- Treat audit JSONL as the operational source of truth.
- Version every externally consumed contract.
- Add deprecation shims only with a removal milestone.
- Avoid re-embedding core daemon logic in Python once Rust service parity is reached.
