# Hardware — Pourquoi GB10 Blackwell

## La machine

| Spec | Valeur |
|---|---|
| **Modèle** | NVIDIA DGX Spark |
| **GPU** | GB10 Blackwell |
| **Mémoire unifiée** | 128 GB |
| **Architecture CPU** | aarch64 (ARM) |
| **TDP** | ~100 W |
| **OS** | Ubuntu 24.04 |
| **CUDA** | 13.0 |
| **PyTorch** | 2.9.1+cu130 |
| **Python** | 3.11 (env Isaac) / 3.12 (système) |
| **Réseau** | Ethernet 10 Gbit |
| **IP locale** | <LAN_IP_SPARK> |

---

## Pourquoi GB10 et pas H100

Question légitime : pourquoi pas un cluster H100 cloud, ou un DGX H100 sur site ?

**Économie d'itération.** Un H100 cloud à l'usage coûte ~3 USD/h en stable, plus en pic. Un cycle d'entraînement RL utile (1500 itérations PPO sur Go2) tourne ~6h. À cinq cycles par semaine, c'est 360 USD/mois minimum, sans compter le stockage, la bande passante, et le temps perdu à provisionner.

Un DGX Spark : achat unique, électricité négligeable, **disponible 24/7 sans frais marginaux**. L'itération devient gratuite. C'est ça qui change la donne pour un développeur indépendant — pas la performance brute.

**Suffisance technique.** GB10 a la VRAM (128 GB unifiée) pour Isaac Sim 5.1 + Isaac Lab 2.3 + plusieurs policies en parallèle. Pour des entraînements à très grande échelle (training fondamentaux GR00T-class), il faut effectivement un cluster. Mais pour **fine-tuner, déployer, mesurer, itérer** sur des policies existantes — le GB10 est largement suffisant.

**Mobilité.** 100W, refroidissement passif, format compact. Le robot peut être amené à la machine, ou la machine au robot. Aucun lien physique avec un datacenter.

---

## Pourquoi pas un Jetson

Le Jetson Orin Nano est l'option naturelle pour de l'embarqué temps réel. Il en a sa place dans le système final (futur Compagnon).

Pour le **développement** — entraînement, simulation, benchmarking — le Jetson est trop limité. Pas assez de VRAM pour Isaac Sim, throughput insuffisant pour itérer plusieurs policies, écosystème Python plus contraint.

Le couple **GB10 (dev) ↔ Jetson (deploy)** est cohérent : tout ce qui est entraîné sur GB10 est exporté en JIT/ONNX, déployable ensuite sur Jetson embarqué. Pour l'instant, le Go2 est piloté par un iPhone embarqué (cf. article principal Nuage), pas un Jetson — décision pragmatique liée aux capacités natives iOS (LiDAR, ARKit, Vision, microphones).

---

## Surprises aarch64

L'écosystème ML aarch64 est mature mais pas universel. Quelques découvertes :

- **PyTorch CUDA aarch64** : disponible (`torch 2.9.1+cu130`). Aucun problème.
- **NVIDIA WebRTC** : pas de package ARM officiel pour LiveStream. Bypass via web viewer custom.
- **Nemotron VLM 12B** : bloqué par `mamba-ssm` non compilé pour aarch64 — fallback sur Groq Llama-4-Scout puis migration GPT-4o.
- **Isaac Sim 5.x** : ARM supporté depuis la 5.0 — **5.1** est la première version pleinement opérationnelle sur ce hardware.

À noter pour qui se lance : prévoir 1 à 2 jours de débogage de dépendances dans les premières semaines.

---

## Le hub Mac M3 Ultra

Le DGX Spark n'est pas seul. Un Mac M3 Ultra (<LAN_IP_HUB>) sert de hub principal :
- Portail Panda (Kanban, API, monitoring) sur port 3010.
- Orchestration multi-agents (sessions tmux).
- Stockage des assets, documentation, logs centralisés.
- Connexion ssh / scp vers Spark pour déploiement.

Cette répartition CPU-puissant (Mac) + GPU-dédié (Spark) couvre l'essentiel sans cluster.

---

## Coût total

À titre indicatif et sans dramatiser :

| Poste | Ordre de grandeur |
|---|---|
| DGX Spark | 4 000 CHF |
| Mac M3 Ultra | 8 000 CHF |
| Unitree Go2 (Edu) | 5 500 CHF |
| Unitree G1 (Education Pro) | 16 000 CHF |
| Apple Vision Pro | 4 000 CHF |
| iPhone 16 Pro Max | 1 500 CHF |
| **Total** | **~39 000 CHF** |

À comparer au coût d'un mois de salaire d'une équipe de 12 ingénieurs ML, ou au TCO annuel d'un cluster H100 dédié.

Ce n'est pas une compétition. C'est une démonstration que **le ticket d'entrée en robotique cognitive est tombé d'un ordre de grandeur** — et que ce changement va débloquer des projets que les grands acteurs n'auraient jamais financés.
