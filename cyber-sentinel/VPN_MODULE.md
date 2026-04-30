# Module VPN — WireGuard self-hosted

> Sub-projet livre le 30 avril 2026. Preuve concrete d'execution rapide en mode agentique.

---

## Contexte

Tailscale a casse mon LAN (extension reseau persistante macOS qui prend le controle des routes meme apres deconnexion). J'avais besoin d'un acces VPN simple au LAN depuis l'iPhone. WireGuard self-hosted etait l'option propre : leger, kernel-supported, app officielle iOS, pas de dependance cloud.

Plutot que d'utiliser un service tiers, j'ai voulu valider que ma pipeline d'execution agentique pouvait livrer un module complet en quelques heures. Le module VPN sert aussi a Sentinel a long terme (acces remote securise pour le dashboard de monitoring).

---

## Decoupage en 6 blocks

Plan agentique livre le matin du 30 avril.

| Block | Scope | Statut |
|-------|-------|--------|
| A | Setup serveur WireGuard sur le hub M3 | livre |
| B | Configuration reseau (subnet 10.66.0.0/24, port 51820 UDP, split-tunnel) | livre |
| C | Scripts CLI gestion clients (add/revoke/list) avec QR codes | livre |
| D | Tests connectivite iPhone bout-en-bout | valide en prod |
| E | API HTTPS Bun TypeScript pour gestion via NestorMobile | livre |
| F | Documentation procedures + backup chiffre vers serveur Linux | livre |

---

## Composants livres

### 11 scripts CLI bash

Tous idempotents, avec self-elevate pattern (`[[ $EUID -ne 0 ]] && exec sudo "$0" "$@"`), defense en profondeur (validation prealable, backup, rollback automatique en cas d'erreur).

- `install-server.sh` / `uninstall-server.sh` — setup serveur WireGuard avec LaunchDaemon
- `install-launchagent.sh` / `uninstall-launchagent.sh` — persistance API
- `add-client.sh <name>` — genere cles + config + QR PNG
- `revoke-client.sh <name>` — revocation avec archive forensics timestamped
- `list-clients.sh` — etat clients + handshake + traffic RX/TX (mode `--json` pour API)
- `diag-handshake.sh [name]` — diagnostic 6 sections couvrant les hypotheses de panne courantes
- `apply-fix-path.sh` — reload propre du LaunchDaemon apres modif plist
- `backup-config.sh` — backup chiffre quotidien vers serveur Linux (age + rsync)
- `setup-sudoers.sh` — installe NOPASSWD scope minimal avec `visudo -c` + cross-check tree + rollback
- `test-server-up.sh` — 11 health checks (interface UP, listener UDP, plist, IP forwarding, pf rules, etc.)
- `restart-server.sh` — wrapper launchctl avec audit log et downtime estimate

### API HTTPS Bun TypeScript (port 3403)

Stack : Bun + Hono. TLS self-signed (RSA 4096, SAN IP+DNS).

5 endpoints :

| Endpoint | Description |
|----------|-------------|
| `GET /api/vpn/health` | Public, healthcheck |
| `GET /api/vpn/status` | Etat serveur + stats clients agreges |
| `GET /api/vpn/clients` | Liste detaillee clients actifs et revoques |
| `GET /api/vpn/clients/:name` | Detail client + config + QR base64 + stats |
| `POST /api/vpn/clients` | Creer client (regex stricte sur name + endpoint) |
| `DELETE /api/vpn/clients/:name` | Revoque client |
| `POST /api/vpn/server/restart` | Redemarrage serveur avec audit log |

Securite par couches :

- Bearer token avec comparaison constant-time (anti timing attack)
- CORS strict 3 origines (`capacitor://localhost`, `https://localhost`, `http://hub-ip:3010`) — pas de wildcard
- Validation regex stricte sur tous les inputs (anti path traversal, anti shell injection)
- `Bun.spawn` avec args[] (jamais d'interpolation de string dans les commandes shell)
- `ExecError` typed → 503 server_unreachable, 500 revoke_failed
- Stderr leak limite a 200 chars dans la reponse client (pas de leak interne)
- Audit log JSON-Lines append-only sur les mutations (chmod 600)

### Tests

11 health checks serveur + 11 curl tests API (dont 4 tests d'attaque bloques) :

| Attaque | Reponse |
|---------|---------|
| GET `/clients/..%2Fetc%2Fpasswd` | 400 invalid_name |
| POST `{endpoint:"; rm -rf /"}` | 400 invalid_endpoint |
| POST nom existant | 409 conflict |
| DELETE deja revoque | 404 not_found |

Validation persistance : `kill -9 PID API` → respawn automatique en 500ms (LaunchAgent `KeepAlive=true`).

### Sudoers NOPASSWD scope minimal

Pour eviter les prompts password lors de l'usage quotidien (admin via tmux ou app mobile), un fichier `/etc/sudoers.d/cyberdefense-vpn` whiteliste 11 paths absolus :

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

Aucun wildcard. `visudo -c -f` validation prealable + cross-check sur le tree complet apres install. Rollback automatique en cas d'invalidite. Le risque de bricker `sudo` = zero (teste deux fois en conditions reelles).

---

## Decouverte non-anticipee

Pendant l'audit systematique d'un bug de mapping macOS userland (l'alias `wg0 → utun6` n'est pas resolu automatiquement par `wg syncconf`), le DEV a decouvert un **bug pre-existant dans le parser de `list-clients.sh`** : le script lisait 9 colonnes alors que `wg show <iface> dump` n'en sort que 8 (la colonne `iface` n'apparait que pour `wg show all dump`). Les variables etaient decalees, le filtre `[iface == wg0]` ne matchait jamais. Affichage `0B/never` meme avec un tunnel actif.

Bug latent depuis l'ecriture initiale du script. Personne ne l'aurait jamais trouve sans cet audit profond. C'est un exemple typique du benefice de la discipline d'audit systematique.

---

## Documentation

| Fichier | Contenu |
|---------|---------|
| `vpn-module/docs/procedures.md` | 270 lignes : usage, troubleshooting, recovery, rotation des cles, section 3.5 sur le piege macOS userland |
| `docs/vpn-architecture.md` | Architecture reseau, plan d'adressage, decisions de securite |
| `docs/vpn-api-spec.md` | Specifications de l'API HTTPS, decisions Nestor STRAT, plan d'implementation |

---

## Statistiques de livraison

- **Demarre** : 30 avril 2026, ~10h45 CET
- **Tunnel iPhone valide bout-en-bout** : meme jour, en quelques heures
- **API HTTPS production** : meme jour, ~14h00 CET
- **Total** : un sprint de demi-journee pour un module fonctionnel et securise

Ce n'est pas Sentinel lui-meme. C'est la preuve que la pipeline d'execution agentique fonctionne sur un perimetre bien defini, avec des decisions architecturales solides et de la rigueur a chaque couche.
