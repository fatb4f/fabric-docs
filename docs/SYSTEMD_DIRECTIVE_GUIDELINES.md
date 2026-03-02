# systemd Directive Guidelines (Daemon + CLI)

This guide defines the current baseline for fabric services (`spawnd`, `meshd`) and can be reused for new daemon/CLI pairs.

## 1) Baseline Service Unit (starting point)

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp daemon
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
NotifyAccess=main
ExecStart=/usr/bin/myappd --config /etc/myapp/config.yaml

# Reliability
Restart=on-failure
RestartSec=2s
TimeoutStartSec=30s
TimeoutStopSec=30s
StartLimitIntervalSec=60s
StartLimitBurst=5

# Logging
SyslogIdentifier=myappd

# Managed directories
StateDirectory=myapp
CacheDirectory=myapp
RuntimeDirectory=myapp

# Baseline least privilege
DynamicUser=yes
NoNewPrivileges=yes

[Install]
WantedBy=multi-user.target
```

## 2) Directive Notes

- `Type=notify` + `NotifyAccess=main`:
  - Use only when daemon emits `READY=1` via `sd_notify`.
  - If not implemented yet, use `Type=simple` and remove notify expectations.
- `StateDirectory=` / `CacheDirectory=` / `RuntimeDirectory=`:
  - Prefer these over manual `mkdir/chown`; they are lifecycle-safe and deterministic.
- Restart/timeouts:
  - Keep explicit values for predictable behavior across hosts.
  - Add `StartLimit*` to prevent restart storms during bad deploys.

## 3) Hardening (incremental)

Add these in controlled steps and validate each change:

```ini
# Filesystem/process isolation
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes

# Kernel/device surface
PrivateDevices=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes

# Capabilities
CapabilityBoundingSet=
AmbientCapabilities=

# Network families (example)
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
```

Guideline: enforce minimum privilege that still preserves required behavior.

## 4) Socket Activation (optional, recommended for CLI -> daemon)

```ini
# /etc/systemd/system/myapp.socket
[Unit]
Description=MyApp control socket

[Socket]
ListenStream=/run/myapp/myapp.sock
SocketMode=0660
SocketUser=root
SocketGroup=myapp

[Install]
WantedBy=sockets.target
```

Adoption modes:
- Prototype mode: daemon binds socket directly (simpler).
- systemd-native mode: daemon accepts systemd-passed FDs (socket activation).

## 5) Reload and Watchdog

- Reload:
  - Define `ExecReload=` and implement safe config reload.
  - Invalid reload input must not crash or deadlock the daemon.
- Watchdog:
  - Use only when heartbeat timing is reliable under load.
  - Configure `WatchdogSec=` and emit watchdog pings from daemon.

## 6) Prototype-Safe Minimum

For first deploy:
- `Type=notify` (only if implemented) or `Type=simple`
- `Restart=on-failure`
- `TimeoutStartSec=` and `TimeoutStopSec=`
- `StateDirectory=` and `RuntimeDirectory=`
- `DynamicUser=yes`
- `NoNewPrivileges=yes`

Defer stricter hardening until runtime behavior is stable and tested.

## 7) Fabric Application Notes

- `spawnd` and `meshd` should share directive policy unless a hard runtime need differs.
- CLI (`spawnctl`/`meshctl`) should target daemon IPC; avoid direct privileged actions.
- Keep unit defaults in dotfiles and treat overrides as host-specific policy, not product defaults.

## References
- `sd_notify`: https://www.freedesktop.org/software/systemd/man/sd_pid_notify_with_fds.html
- `systemd.exec`: https://www.freedesktop.org/software/systemd/man/systemd.exec.html
- `systemd.socket`: https://www.freedesktop.org/software/systemd/man/systemd.socket.html
