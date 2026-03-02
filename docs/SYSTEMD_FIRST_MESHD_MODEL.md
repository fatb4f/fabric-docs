# systemd-first control model for meshd

This document refines a systemd-first runtime model for `mesh` (compatibility aliases: `watcher`, `watchctl`).

## 1) Daemon as a user service

`~/.config/systemd/user/meshd-daemon.service`

```ini
[Unit]
Description=meshd event router (meshctl daemon)
After=default.target
PartOf=meshd.target

[Service]
Type=simple
ExecStart=%h/.local/bin/meshctl daemon
Restart=always
RestartSec=1
EnvironmentFile=-%h/.config/mesh/mesh.env
WorkingDirectory=%h
SyslogIdentifier=meshd-daemon

[Install]
WantedBy=meshd.target
```

Notes:
- Keep command line short.
- Prefer env vars (`MESH_CONFIG`, `MESH_STATE_DIR`, `MESH_AUDIT_DIR`).

## 2) Sources as templated user services

`~/.config/systemd/user/meshd-source@.service`

```ini
[Unit]
Description=meshd source (%i)
After=default.target
PartOf=meshd.target

[Service]
Type=simple
EnvironmentFile=-%h/.config/mesh/mesh.env
ExecStart=%h/.local/bin/meshctl daemon \
  --profiles ${MESH_CONFIG:-%h/.config/mesh/profiles.yaml} \
  --source-command "%h/.local/bin/%i"
Restart=always
RestartSec=2
SyslogIdentifier=meshd-source-%i

[Install]
WantedBy=meshd.target
```

Why this shape:
- It matches current `watchctl/meshctl` behavior (`daemon --source-command ...`).
- No new `handle-event --stdin` CLI is required.

## 3) Profiles as systemd enable/disable markers

`~/.config/systemd/user/meshd-profile@.service`

```ini
[Unit]
Description=meshd profile enabled marker (%i)

[Service]
Type=oneshot
ExecStart=/usr/bin/true
RemainAfterExit=yes
```

Usage:
- enable profile: `systemctl --user enable meshd-profile@vps_secops.service`
- disable profile: `systemctl --user disable meshd-profile@vps_secops.service`
- check profile: `systemctl --user is-enabled meshd-profile@vps_secops.service`

Daemon behavior requirement:
- before running matched profile, check `is-enabled meshd-profile@<profile>.service`
- if disabled, log `skipped_disabled_profile`
- if enabled, proceed

## 4) Per-run execution as transient units

For TUI visibility and isolation, execute each profile run via transient unit:

```bash
systemd-run --user --collect \
  --unit="meshd-run@${PROFILE}-${EVENT_ID}" \
  --property=Environment=WATCH_EVENT_FILE=${EVENT_FILE} \
  %h/.local/bin/meshctl run-profile "${PROFILE}" --event-file "${EVENT_FILE}"
```

This keeps `meshctl` as audit authority while systemd provides:
- per-run unit visibility
- cgroup/resource isolation
- run-scoped journald streams

## 5) One-command ops flow

```bash
systemctl --user daemon-reload
systemctl --user enable --now meshd.target
systemctl --user enable --now meshd-daemon.service
systemctl --user enable --now meshd-source@codex-event-source.service
systemctl --user enable meshd-profile@vps_secops.service
```

Views:

```bash
systemctl --user list-units 'meshd*'
journalctl --user-unit meshd-daemon -f
journalctl --user-unit 'meshd-source@codex-event-source' -f
```

## 6) Optional safety controls for transient runs

Apply per-profile `systemd-run` properties, e.g.:
- `NoNewPrivileges=yes`
- `PrivateTmp=yes`
- `MemoryMax=500M`
- `CPUQuota=50%`

Recommended:
- store per-profile runtime limits in config
- map to `systemd-run --property=...` at dispatch time

## 7) Minimal implementation checklist

1. Add profile-enable gate against `meshd-profile@<profile>.service`.
2. Add transient execution mode for `run-profile` using `systemd-run`.
3. Add `--event-file` support for `run-profile` (for deterministic dispatch).
4. Ship unit templates and optional installer helper.
5. Keep compatibility aliases during migration:
   - `watcher-*` -> `meshd-*`
   - `watchctl` -> `meshctl`
