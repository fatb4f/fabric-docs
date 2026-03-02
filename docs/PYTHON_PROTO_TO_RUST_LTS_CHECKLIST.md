# Python Prototype -> Rust LTS Daemon + CLI Checklist

This is a normalized checklist for systemd-first services where Python is the prototype substrate and Rust is the long-term daemon/runtime target.

## 1) Product Shape and Invariants
- [ ] Define one stable service/CLI pair (example: `spawnd` + `spawnctl`, `meshd` + `meshctl`).
- [ ] Freeze command contract now: flags, exit codes, JSON schema, human output.
- [ ] Freeze daemon IPC contract now and version it (`protocol_version`).
- [ ] Select primary transport:
  - [ ] Unix domain socket (recommended).
  - [ ] Optional later: systemd socket activation.
- [ ] Declare supported runtime modes:
  - [ ] Linux + systemd (primary).
  - [ ] Non-systemd foreground mode for dev/CI.

## 2) Daemon/CLI Boundary
- [ ] CLI never performs privileged operations directly; daemon owns execution.
- [ ] Define request/response protocol with:
  - [ ] versioned envelope.
  - [ ] machine output (`--json`) + human output.
  - [ ] timeout/retry semantics.
- [ ] Provide foreground mode (`*d --foreground`) for debug and integration tests.
- [ ] Define lifecycle interface:
  - [ ] `start|stop|restart|status|reload`.
  - [ ] optional one-shot dispatch (`run`).
- [ ] Define readiness + health contract:
  - [ ] readiness condition.
  - [ ] `health` command/API.

## 3) systemd Unit Requirements

Reference: `docs/SYSTEMD_DIRECTIVE_GUIDELINES.md` for directive baseline, hardening tiers, and socket-activation patterns.

### 3.1 Service Units
- [ ] Provide canonical unit files:
  - [ ] system unit (`myapp.service`) and/or user unit.
  - [ ] optional templated units (`myapp@.service`).
- [ ] Keep dependencies explicit/minimal (`After=`, `Wants=`, `Requires=`).
- [ ] Set service behavior:
  - [ ] `ExecStart=` points to stable installed path.
  - [ ] `Restart=` policy and `RestartSec=`.
  - [ ] `TimeoutStartSec=` / `TimeoutStopSec=`.
  - [ ] `ExecReload=` and reload handling.
- [ ] Logging model:
  - [ ] stdout/stderr to journald by default.
  - [ ] log level configurable.
- [ ] Optional systemd-native features:
  - [ ] `Type=notify` + `sd_notify(READY=1)`.
  - [ ] watchdog heartbeat + `WatchdogSec=`.

### 3.2 Hardening Baseline
- [ ] Use dedicated service identity (`User=`/`Group=` or `DynamicUser=yes`).
- [ ] Enable baseline restrictions:
  - [ ] `NoNewPrivileges=yes`
  - [ ] `PrivateTmp=yes`
  - [ ] `ProtectSystem=` (`full` or `strict`)
  - [ ] `ProtectHome=` where feasible
  - [ ] `CapabilityBoundingSet=` minimal/empty
  - [ ] `RestrictAddressFamilies=` minimal set
- [ ] Prefer systemd-managed dirs:
  - [ ] `StateDirectory=`
  - [ ] `CacheDirectory=`
  - [ ] `RuntimeDirectory=`
  - [ ] optional `ConfigurationDirectory=`

## 4) Config, State, and XDG Paths
- [ ] Define precedence once and document it:
  - [ ] CLI flags > env > config file > defaults.
- [ ] Support XDG for user mode:
  - [ ] config: `~/.config/<app>/`
  - [ ] state: `~/.local/state/<app>/`
  - [ ] cache: `~/.cache/<app>/`
- [ ] Support system paths for system mode:
  - [ ] config: `/etc/<app>/`
  - [ ] state: `/var/lib/<app>/`
  - [ ] cache: `/var/cache/<app>/`
  - [ ] runtime: `/run/<app>/`
- [ ] Define safe reload behavior:
  - [ ] invalid new config must not brick running service.

## 5) Packaging and Install Paths
- [ ] Choose first-class distribution path (at least one):
  - [ ] distro package (`.pkg.tar.zst`, `.deb`, `.rpm`) for service deployment.
  - [ ] Python package (`pyproject.toml`) for prototype/dev installs.
  - [ ] Rust release binaries for LTS rollout.
- [ ] Package must include:
  - [ ] daemon binary/script
  - [ ] CLI
  - [ ] systemd unit files
  - [ ] default config template
- [ ] Define install/uninstall policy:
  - [ ] whether services auto-start post-install
  - [ ] whether config/state are preserved on uninstall

## 6) Operations and Reliability
- [ ] Clean SIGTERM shutdown within timeout.
- [ ] No stale PID/runtime artifacts.
- [ ] Concurrency/backpressure model documented.
- [ ] Resource constraints defined (systemd + internal limits).
- [ ] Upgrade compatibility policy:
  - [ ] state/schema migration versioning
  - [ ] rollback expectations
- [ ] Security controls:
  - [ ] CLI->daemon authz model (single-user vs multi-user)
  - [ ] socket ownership/mode policy
  - [ ] strict input validation

## 7) Testing Gates
- [ ] Unit tests for core logic.
- [ ] Integration tests (foreground daemon + CLI).
- [ ] Contract tests for JSON output + exit-code stability.
- [ ] At least one systemd integration test (`systemd-run`/container VM).
- [ ] Packaging smoke test on clean host/container.

## 8) CI/CD and Release
- [ ] PR CI gates:
  - [ ] lint/format
  - [ ] unit + integration tests
  - [ ] build artifacts
- [ ] Tag release flow:
  - [ ] version discipline
  - [ ] release notes/changelog
  - [ ] signed checksums for artifacts

## 9) Rust LTS Cutover Plan
- [ ] Freeze interfaces before cutover:
  - [ ] CLI UX
  - [ ] daemon IPC
  - [ ] config schema
  - [ ] state layout
- [ ] Maintain one golden integration suite for Python and Rust implementations.
- [ ] Decide phased replacement strategy:
  - [ ] daemon first
  - [ ] CLI first
  - [ ] both together with protocol stability
- [ ] Maintain observability parity (logs/health/metrics).

## 10) Minimum Ship Definition
- [ ] Daemon runs under systemd with deterministic restart and shutdown behavior.
- [ ] Status/health is queryable through CLI.
- [ ] Config/state paths are correct and documented.
- [ ] One clean-machine install path is CI-verified.
- [ ] Upgrade path preserves state or explicitly documents breaking behavior.

## Fabric-Specific Notes
- Apply this checklist to both service pairs:
  - `spawnd` + `spawnctl`
  - `meshd` + `meshctl`
- Keep the control contract stable while implementation language changes.
- Treat the daemon protocol + config schema as the long-lived compatibility surface.
