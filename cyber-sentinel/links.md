# Liens — code, screenshots, documents

> Pour la mise en page sur GitHub Pages ou pour integration dans le repo principal GitHub Vision.

---

## Documents de reference (dans le repo CyberDefense)

| Lien interne | Description |
|--------------|-------------|
| `docs/prd.md` | Product Requirements Document complet |
| `docs/masterplan-phase0.md` | Plan de la Phase 0 — recherche & etat de l'art |
| `docs/research-phase0-axes1-2.md` | Rapport de synthese : 10 outils + 16 APIs |
| `docs/competitive-analysis-edr.md` | Analyse detaillee CrowdStrike, SentinelOne, Defender |
| `docs/threat-intel-apis-research.md` | Inventaire detaille des 16 sources de threat intelligence |
| `docs/sentinel-vue-densemble.html` | Document HTML pedagogique pour audience non-technique |
| `docs/vpn-architecture.md` | Architecture du module VPN |
| `docs/vpn-api-spec.md` | Specifications de l'API HTTPS du module VPN |
| `docs/linkedin-article.md` | Article LinkedIn 30 avril 2026 |

---

## Code livre (dans le repo CyberDefense)

### Module VPN — `vpn-module/`

```
vpn-module/
├── api/
│   ├── src/
│   │   ├── index.ts          # Bun.serve TLS + middleware chain
│   │   ├── auth.ts           # Bearer middleware constant-time
│   │   ├── cors.ts           # CORS strict 3 origines
│   │   ├── routes/
│   │   │   ├── health.ts     # GET /api/vpn/health (public)
│   │   │   ├── status.ts     # GET /api/vpn/status (auth)
│   │   │   ├── clients.ts    # GET/POST/DELETE /api/vpn/clients[/:name]
│   │   │   └── server.ts     # POST /api/vpn/server/restart
│   │   └── lib/
│   │       └── exec.ts       # Bun.spawn args[] wrapper
│   ├── certs/                # TLS self-signed (chmod 600, .gitignore)
│   ├── logs/                 # api.{out,err} + audit.log
│   ├── .env.example
│   └── package.json
├── configs/
│   ├── wg0.conf.template
│   ├── client.conf.template
│   ├── launchdaemons/org.cyberdefense.wireguard.plist
│   ├── launchagents/com.cyberdefense.vpn-api.plist
│   └── pf.anchors/wireguard
├── scripts/
│   ├── lib/
│   │   ├── common.sh         # helpers + reload_wireguard()
│   │   └── ip-allocator.sh   # next free IP in subnet
│   ├── install-server.sh
│   ├── uninstall-server.sh
│   ├── install-launchagent.sh
│   ├── uninstall-launchagent.sh
│   ├── add-client.sh
│   ├── revoke-client.sh
│   ├── list-clients.sh
│   ├── diag-handshake.sh
│   ├── apply-fix-path.sh
│   ├── backup-config.sh
│   ├── setup-sudoers.sh
│   └── restart-server.sh
├── tests/
│   └── test-server-up.sh     # 11 health checks
├── docs/
│   ├── README.md
│   └── procedures.md         # 270 lignes
├── keys/                     # server pub/priv + clients/ (chmod 700)
└── .gitignore
```

### POC Phase 0 — `CyberDefense APP/poc-axe4/`

```
poc-axe4/
├── results/
│   ├── benchmark-report.md
│   ├── benchmark-events.json   # 100 events de test
│   ├── metrics.json
│   ├── results-haiku-4.json    # 110 KB
│   └── results-sonnet-4.json   # 173 KB
├── src/
│   ├── generator.ts            # Genere 100 events
│   ├── benchmark.ts            # Roule les events sur les modeles
│   └── prompts/                # Templates prompt LLM
└── package.json
```

---

## Captures d'ecran a integrer

### Article LinkedIn — visuels recommandes

| # | Image | Description |
|---|-------|-------------|
| 1 | Le diagramme des 3 couches | Capture de la section Architecture du document HTML, fond sombre, jaune accent |
| 2 | Le tableau de comparaison | Capture du tableau "Programme standard vs Sentinel" avec les 8 criteres |
| 3 | Le scenario kito RAT | Capture de la zone "Verdict Sentinel" avec le raisonnement en pseudo-code |
| 4 | Stats POC | Les 4 chiffres cles : 100% recall, 97.6% confiance, 2.64s, $1.52/jour |

### GitHub Vision — visuels recommandes

| # | Image | Description |
|---|-------|-------------|
| 1 | Header vitrine | "CyberDefense / Sentinel" avec le tagline "Detection par raisonnement, pas par signature" |
| 2 | Architecture 3 couches | Diagramme propre, fond sombre |
| 3 | Tableau comparatif | Les 8 criteres standard vs Sentinel |
| 4 | Tableau detection kito | EDR commerciaux ~30-65% vs Sentinel >90% |
| 5 | Capture API VPN | curl test avec reponse JSON propre |
| 6 | Liste scripts | Arborescence vpn-module/scripts/ |

---

## Acces public

Le code et les rapports techniques seront ouverts au fur et a mesure de la livraison. La strategie de publication :

- **Phase 0** (recherche + POC) : rapports markdown publics maintenant ; code POC public sur GitHub
- **Module VPN** : code public sur GitHub apres validation finale (prevue cette semaine)
- **Phase 1** (daemon + Cortex) : code public au fur et a mesure des sprints

Licence prevue : Apache 2.0 pour le code, CC-BY-SA 4.0 pour la documentation.

---

## Contact

Le projet est mene en parallele d'autres initiatives de la ferme InfinityCloud. Les retours techniques (architecture, choix des LLM, threat intel sources, methodologie agentique) sont tres bienvenus — directement sur le repo, ou via les canaux LinkedIn habituels.
