# Vision Robotique — Infinity Cloud

> Robotique cognitive en environnement indépendant — depuis la Suisse, sur du matériel grand public NVIDIA.

*Sous-projet de [Vision](../README.md) — vitrine technique du programme robotique.*

[![Hardware](https://img.shields.io/badge/Hardware-NVIDIA%20DGX%20Spark%20GB10-76b900)](docs/hardware.md)
[![Sim](https://img.shields.io/badge/Sim-Isaac%20Sim%205.1-76b900)](docs/architecture.md)
[![Robot](https://img.shields.io/badge/Robot-Unitree%20Go2%20%2B%20G1-blue)](docs/architecture.md)
[![Country](https://img.shields.io/badge/Made%20in-Switzerland-red)](#)

---

## Pourquoi ce repo

Ce repo documente une infrastructure de robotique cognitive bâtie en mode **indépendant** depuis la Suisse — pas par une équipe de douze ingénieurs, pas sur un cluster cloud à six chiffres, pas avec une roadmap dictée par un board.

Une seule personne, un **NVIDIA DGX Spark (GB10 Blackwell, ~100W)** posé sur un bureau, un Mac M3 Ultra comme hub, et la conviction que la robotique d'assistance est trop importante pour être laissée aux seuls grands groupes.

Ce que vous trouverez ici :
- Les **résultats reproductibles** (logs, métriques, scripts).
- L'**architecture** (3 couches : cerveau / pont / monde).
- Le **matériel** (pourquoi GB10, pourquoi pas un H100, pourquoi pas un Jetson).
- La **méthodologie** (simulation-first, discipline d'itération, ICSD).
- La **roadmap** (où ça va, et ce qui reste honnêtement à prouver).

---

## En 30 secondes

| Brique | État | Lien |
|---|---|---|
| **Inférence GR00T N1.7 sur Unitree G1** | Walk + turn validés | [docs/results.md#groot-n17](docs/results.md) |
| **WBC clone GEAR-SONIC** | Référence bas niveau | [docs/results.md#wbc](docs/results.md) |
| **Pipeline Hy3D → Isaac Sim** | 3 GLB livrés | [docs/results.md#hy3d](docs/results.md) |
| **Patrouille Go2 + VLM (GPT-4o)** | E2E validé | [docs/results.md#patrol](docs/results.md) |
| **PID yaw correction** | Drift 70% → 6% | [docs/results.md#pid](docs/results.md) |
| **Apple Vision Pro cockpit** | TDD v2.0 | [docs/architecture.md#cockpit](docs/architecture.md) |

---

## Le pari : 100W contre des mégawatts

Un cluster d'entraînement RL classique consomme l'équivalent d'une commune de 5 000 habitants. Le DGX Spark consomme moins qu'un sèche-cheveux.

Cette différence n'est pas qu'idéologique. Elle change ce qu'on peut **se permettre d'itérer**.

- 128 GB de mémoire unifiée — Isaac Sim 5.1 + Isaac Lab 2.3 + plusieurs policies en parallèle, sans swap.
- Architecture aarch64 (ARM) — l'écosystème ML est mature, avec quelques angles morts à connaître.
- Refroidissement passif — sessions de plusieurs heures sans throttling.
- 100W TDP — branchable sur n'importe quelle prise standard, transportable, silencieux.

Pas un substitut au DGX H100 d'un grand groupe. Un **outil suffisant pour boucler un cycle complet** : simulation → policy → robot → mesure → itération.

---

## Structure

```
robotics/
├── README.md                 ← vous êtes ici
├── docs/
│   ├── linkedin-article.md   ← article public
│   ├── architecture.md       ← 3 couches : cerveau / pont / monde
│   ├── hardware.md           ← GB10 Blackwell, choix techniques
│   ├── results.md            ← métriques, screenshots, logs
│   ├── methodology.md        ← simulation-first, ICSD, itération
│   └── roadmap.md            ← prochains jalons, sim-to-real
├── media/                    ← captures Isaac Sim, viewer
└── research/
    ├── compatibility.md      ← walk-these-ways, unitree_rl_gym, GR00T
    └── vlm_benchmark.md      ← Groq vs GPT-4o vs qwen3-vl vs Nemotron
```

---

## Pour qui

- **Ingénieurs robotique** qui veulent voir ce que produit un développeur seul équipé d'un GB10.
- **Chercheurs VLA / VLM** qui s'intéressent à la discrimination visuelle en environnement simulé.
- **Décideurs santé / silver economy** qui cherchent des partenaires pour l'assistance aux personnes vulnérables.
- **Développeurs indé** qui hésitent à se lancer en robotique sans cluster cloud — voici une preuve que c'est possible.

---

## Liens

- **Article LinkedIn** : [Bâtir une infrastructure de robotique cognitive depuis la Suisse — seul, avec un GB10 Blackwell sous le bureau](docs/linkedin-article.md)
- **Repo principal RoboticProgramAI** : [github.com/infinitycloud-ch/roboticprogramai](https://github.com/infinitycloud-ch/roboticprogramai)
- **Contact** : Minh-Tam Dang — Infinity Cloud Sàrl, Suisse

---

*Dernière mise à jour : 2026-04-30 — itéré toutes les semaines au rythme des breakthroughs.*
