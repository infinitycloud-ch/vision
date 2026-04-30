# Links — code, screenshots, documents

> For GitHub Pages layout or for integration into the main GitHub Vision repo.

---

## Reference documents (in the CyberDefense repo)

| Internal link | Description |
|---------------|-------------|
| `docs/prd.md` | Complete Product Requirements Document |
| `docs/masterplan-phase0.md` | Phase 0 plan — research & state of the art |
| `docs/research-phase0-axes1-2.md` | Synthesis report: 10 tools + 16 APIs |
| `docs/competitive-analysis-edr.md` | Detailed analysis of CrowdStrike, SentinelOne, Defender |
| `docs/threat-intel-apis-research.md` | Detailed inventory of 16 threat intelligence sources |
| `docs/sentinel-vue-densemble.html` | HTML educational document for non-technical audience |
| `docs/vpn-architecture.md` | VPN module architecture |
| `docs/vpn-api-spec.md` | VPN HTTPS API specifications |
| `docs/linkedin-article-en.md` | LinkedIn article (English) |
| `docs/linkedin-article.md` | LinkedIn article (French) |

---

## Shipped code (in the CyberDefense repo)

### VPN module — `vpn-module/`

```
vpn-module/
├── api/
│   ├── src/
│   │   ├── index.ts          # Bun.serve TLS + middleware chain
│   │   ├── auth.ts           # Bearer middleware, constant-time
│   │   ├── cors.ts           # Strict CORS, 3 origins
│   │   ├── routes/
│   │   │   ├── health.ts     # GET /api/vpn/health (public)
│   │   │   ├── status.ts     # GET /api/vpn/status (auth)
│   │   │   ├── clients.ts    # GET/POST/DELETE /api/vpn/clients[/:name]
│   │   │   └── server.ts     # POST /api/vpn/server/restart
│   │   └── lib/
│   │       └── exec.ts       # Bun.spawn args[] wrapper
│   ├── certs/                # Self-signed TLS (chmod 600, .gitignore)
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
│   └── procedures.md         # 270 lines
├── keys/                     # server pub/priv + clients/ (chmod 700)
└── .gitignore
```

### Phase 0 POC — `CyberDefense APP/poc-axe4/`

```
poc-axe4/
├── results/
│   ├── benchmark-report.md
│   ├── benchmark-events.json   # 100 test events
│   ├── metrics.json
│   ├── results-haiku-4.json    # 110 KB
│   └── results-sonnet-4.json   # 173 KB
├── src/
│   ├── generator.ts            # Generates 100 events
│   ├── benchmark.ts            # Runs events through models
│   └── prompts/                # LLM prompt templates
└── package.json
```

---

## Screenshots to integrate

### LinkedIn article — recommended visuals

| # | Image | Description |
|---|-------|-------------|
| 1 | The 3-layer diagram | Capture from the Architecture section of the HTML document, dark background, yellow accent |
| 2 | The comparison table | Capture of the "Standard vs Sentinel" table with the 8 criteria |
| 3 | The kito RAT scenario | Capture of the "Sentinel verdict" zone with the reasoning in pseudo-code |
| 4 | POC stats | The 4 key numbers: 100% recall, 97.6% confidence, 2.64s, $1.52/day |

### GitHub Vision — recommended visuals

| # | Image | Description |
|---|-------|-------------|
| 1 | Showcase header | "CyberDefense / Sentinel" with the tagline "Detection by reasoning, not by signature" |
| 2 | 3-layer architecture | Clean diagram, dark background |
| 3 | Comparison table | The 8 standard vs Sentinel criteria |
| 4 | Kito detection table | Commercial EDRs ~30-65% vs Sentinel >90% |
| 5 | VPN API capture | curl test with clean JSON response |
| 6 | Scripts list | Tree view of `vpn-module/scripts/` |

---

## Public access

Code and technical reports will be opened progressively as they ship. Publication strategy:

- **Phase 0** (research + POC): markdown reports public now; POC code public on GitHub
- **VPN module**: public code on GitHub after final validation (planned this week)
- **Phase 1** (daemon + Cortex): public code as sprints ship

Planned license: Apache 2.0 for code, CC-BY-SA 4.0 for documentation.

---

## Contact

The project runs in parallel with other InfinityCloud farm initiatives. Technical feedback (architecture, LLM choices, threat intel sources, agentic methodology) is very welcome — directly on the repo, or via the usual LinkedIn channels.
