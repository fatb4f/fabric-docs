# Fabric MUST Rules (spawn + mesh)

These rules are mandatory for `spawn` and `mesh` to avoid script-glue drift.

## 1) Daemon-First Architecture
- CLI MUST orchestrate through daemon IPC.
- CLI MUST NOT execute privileged operations directly.

## 2) Stable Service Contract
- Commands MUST expose stable lifecycle verbs (`start|stop|restart|status|reload` or mapped equivalents).
- Machine output MUST be available (`--json`) with stable schemas.
- Exit-code semantics MUST be documented and versioned.
- Timeout and retry behavior MUST be deterministic.

## 3) Path and Runtime Model
- Runtime paths MUST follow XDG in user mode and systemd directories in system mode.
- Daemons MUST read config from XDG/system paths, not repo-relative paths.
- Daemons MUST write state/cache only to designated runtime directories.

## 4) systemd Baseline
- Services MUST define explicit restart and timeout policy.
- Services MUST include baseline hardening (`NoNewPrivileges`, managed directories, identity model).
- `Type=notify` MUST only be used if `sd_notify` readiness is implemented.

## 5) Config Discipline
- Config load pipeline MUST be: load -> normalize -> validate.
- Validation MUST use typed schema models (pydantic).
- Config and protocol versions MUST be explicit and backward-compatible by policy.

## 6) Test and Merge Gates
- Integration tests MUST cover daemon foreground mode + CLI interactions.
- Contract tests MUST validate JSON output and exit-code behavior.
- No merge without passing lint + tests + integration gates.

## 7) Python -> Rust Migration Safety
- CLI UX, daemon IPC, config schema, and state layout MUST be frozen before cutover.
- A shared golden integration suite MUST pass against both Python and Rust implementations.

## 8) Shell Usage Constraints
- Shell scripts MUST be limited to wrappers, install helpers, or ops glue.
- Core control flow, policy, and protocol logic MUST live in daemon/CLI code, not shell.

## References
- `docs/PYTHON_PROTO_TO_RUST_LTS_CHECKLIST.md`
- `docs/SYSTEMD_DIRECTIVE_GUIDELINES.md`
