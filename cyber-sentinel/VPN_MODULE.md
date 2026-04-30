# VPN module — self-hosted WireGuard

> Sub-project shipped April 30, 2026. Concrete proof of fast execution in agentic mode.

---

## Context

Tailscale broke my LAN (persistent macOS network extension that took control of routes even after disconnection). I needed simple VPN access to the LAN from my iPhone. Self-hosted WireGuard was the clean option: lightweight, kernel-supported, official iOS app, no cloud dependency.

Rather than use a third-party service, I wanted to validate that my agentic execution pipeline could ship a complete module in a few hours. The VPN module also serves Sentinel long-term (secure remote access for the monitoring dashboard).

---

## Six-block breakdown

Agentic plan delivered the morning of April 30.

| Block | Scope | Status |
|-------|-------|--------|
| A | WireGuard server setup on the M3 hub | shipped |
| B | Network configuration (10.66.0.0/24 subnet, port 51820 UDP, split-tunnel) | shipped |
| C | Client management CLI scripts (add/revoke/list) with QR codes | shipped |
| D | iPhone end-to-end connectivity tests | validated in production |
| E | Bun TypeScript HTTPS API for management via NestorMobile | shipped |
| F | Procedures documentation + encrypted backup to Linux server | shipped |

---

## Components delivered

### 11 bash CLI scripts

All idempotent, with self-elevate pattern (`[[ $EUID -ne 0 ]] && exec sudo "$0" "$@"`), defense in depth (preliminary validation, backup, automatic rollback on error).

- `install-server.sh` / `uninstall-server.sh` — WireGuard server setup with LaunchDaemon
- `install-launchagent.sh` / `uninstall-launchagent.sh` — API persistence
- `add-client.sh <name>` — generate keys + config + QR PNG
- `revoke-client.sh <name>` — revocation with timestamped forensics archive
- `list-clients.sh` — client status + handshake + RX/TX traffic (`--json` mode for API)
- `diag-handshake.sh [name]` — 6-section diagnostic covering common failure hypotheses
- `apply-fix-path.sh` — clean LaunchDaemon reload after plist modification
- `backup-config.sh` — daily encrypted backup to Linux server (age + rsync)
- `setup-sudoers.sh` — installs minimal-scope NOPASSWD with `visudo -c` + tree cross-check + rollback
- `test-server-up.sh` — 11 health checks (interface UP, UDP listener, plist, IP forwarding, pf rules, etc.)
- `restart-server.sh` — launchctl wrapper with audit log and downtime estimate

### Bun TypeScript HTTPS API (port 3403)

Stack: Bun + Hono. Self-signed TLS (RSA 4096, SAN IP+DNS).

5 endpoints:

| Endpoint | Description |
|----------|-------------|
| `GET /api/vpn/health` | Public, healthcheck |
| `GET /api/vpn/status` | Server state + aggregated client stats |
| `GET /api/vpn/clients` | Detailed list of active and revoked clients |
| `GET /api/vpn/clients/:name` | Client detail + config + QR base64 + stats |
| `POST /api/vpn/clients` | Create client (strict regex on name + endpoint) |
| `DELETE /api/vpn/clients/:name` | Revoke client |
| `POST /api/vpn/server/restart` | Server restart with audit log |

Defense by layers:

- Bearer token with constant-time comparison (anti timing attack)
- Strict CORS, 3 origins (`capacitor://localhost`, `https://localhost`, `http://hub-ip:3010`) — no wildcard
- Strict regex validation on all inputs (anti path traversal, anti shell injection)
- `Bun.spawn` with args[] (no string interpolation in shell commands)
- Typed `ExecError` → 503 server_unreachable, 500 revoke_failed
- Stderr leak limited to 200 chars in client response (no internal leak)
- Append-only JSON-Lines audit log on mutations (chmod 600)

### Tests

11 server health checks + 11 API curl tests (including 4 blocked attack tests):

| Attack | Response |
|--------|---------|
| GET `/clients/..%2Fetc%2Fpasswd` | 400 invalid_name |
| POST `{endpoint:"; rm -rf /"}` | 400 invalid_endpoint |
| POST existing name | 409 conflict |
| DELETE already revoked | 404 not_found |

Persistence validation: `kill -9 PID API` → automatic respawn in 500ms (LaunchAgent `KeepAlive=true`).

### Minimal-scope NOPASSWD sudoers

To avoid password prompts during daily use (admin via tmux or mobile app), an `/etc/sudoers.d/cyberdefense-vpn` file whitelists 11 absolute paths:

```
<user> ALL=(root) NOPASSWD: /opt/homebrew/bin/wg, \
    /opt/homebrew/bin/wg-quick, \
    /Users/.../scripts/install-server.sh, \
    /Users/.../scripts/uninstall-server.sh, \
    /Users/.../scripts/add-client.sh, \
    /Users/.../scripts/revoke-client.sh, \
    /Users/.../scripts/list-clients.sh, \
    /Users/.../scripts/diag-handshake.sh, \
    /Users/.../scripts/restart-server.sh, \
    /Users/.../scripts/apply-fix-path.sh, \
    /Users/.../scripts/backup-config.sh
```

No wildcards. `visudo -c -f` preliminary validation + cross-check on the full tree after install. Automatic rollback on invalidity. Risk of bricking `sudo`: zero (tested twice in real conditions).

---

## Unanticipated discovery

During systematic audit of a macOS userland mapping bug (the `wg0 → utun6` alias not auto-resolved by `wg syncconf`), the DEV discovered a **pre-existing bug in the `list-clients.sh` parser**: the script was reading 9 columns when `wg show <iface> dump` only outputs 8 (the `iface` column appears only with `wg show all dump`). Variables were shifted, the `[iface == wg0]` filter never matched. `0B/never` displayed even with an active tunnel.

Latent bug since the script's initial writing. Nobody would have ever found it without this deep audit. Classic example of the benefit of systematic audit discipline.

---

## Documentation

| File | Content |
|------|---------|
| `vpn-module/docs/procedures.md` | 270 lines: usage, troubleshooting, recovery, key rotation, section 3.5 on the macOS userland trap |
| `docs/vpn-architecture.md` | Network architecture, addressing plan, security decisions |
| `docs/vpn-api-spec.md` | HTTPS API specifications, Nestor STRAT decisions, implementation plan |

---

## Delivery statistics

- **Started**: April 30, 2026, ~10:45 CET
- **iPhone tunnel validated end-to-end**: same day, in a few hours
- **Production HTTPS API**: same day, ~14:00 CET
- **Total**: a half-day sprint for a functional and secure module

This isn't Sentinel itself. It's evidence that the agentic execution pipeline works on a well-defined scope, with solid architectural decisions and rigor at every layer.
