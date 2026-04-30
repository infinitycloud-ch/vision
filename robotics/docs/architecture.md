# Architecture — Trois couches découplées

## Le principe

L'intelligence d'un robot d'assistance n'est pas monolithique. Elle se décompose en **trois couches** avec des contraintes temporelles et matérielles radicalement différentes. Mélanger ces couches — la tentation de tout mettre dans un seul script — est la première erreur.

```
┌─────────────────────────────────────────────────┐
│  COUCHE 1 — CERVEAU (cognition)                 │
│  Cloud LLM, ~100ms, raisonnement                │
│  GPT-4o, Claude, Groq Llama-4-Scout             │
└──────────────────┬──────────────────────────────┘
                   │ MonoCLI (mémoire persistante)
                   ▼
┌─────────────────────────────────────────────────┐
│  COUCHE 2 — PONT NEURAL (interface)             │
│  ROS2 Jazzy, ~10ms, traduction intention→cmd    │
│  SimAdapter / Go2Adapter                        │
└──────────────────┬──────────────────────────────┘
                   │ UDP / DDS
                   ▼
┌─────────────────────────────────────────────────┐
│  COUCHE 3 — MONDE (locomotion / perception)     │
│  Isaac Sim 5.1 ↔ Go2/G1 réel, <1ms, contrôle    │
│  Policy RL, capteurs, équilibre                 │
└─────────────────────────────────────────────────┘
```

---

## Couche 1 — Le cerveau (cognition + empathie)

**Rôle :** raisonner, planifier, dialoguer, mémoriser.

**Outils :**
- **GPT-4o** pour la vision (description scène, détection humain, identification).
- **Claude** pour le raisonnement long et la planification de mission.
- **Groq Llama-4-Scout** pour les requêtes ultra-rapides (~0.34s, 187 tok/s) quand la latence prime.
- **MonoCLI** — moteur propriétaire de mémoire persistante structurée. Le robot se souvient des interactions, des préférences, des incidents.

**Latence acceptable :** 100ms à plusieurs secondes selon la requête. Le cerveau ne doit jamais bloquer la couche 3.

---

## Couche 2 — Le pont neural (interface)

**Rôle :** traduire une intention cognitive ("approche-toi de la personne et dis bonjour") en commandes robotiques (Twist, Pose).

**Outils :**
- **ROS2 Jazzy** — standard de l'industrie, sur Ubuntu 24.04.
- **SimAdapter / Go2Adapter** — abstraction qui permet de basculer entre simulation et réel sans changer le cerveau.
- **API contract** — 7 commandes atomiques (`sense`, `move`, `turn`, `stand`, `goto`, `look`, `reset`).

**Latence cible :** 10ms. Le pont ne fait pas de RL, pas de calcul lourd — juste de la traduction et de l'orchestration.

---

## Couche 3 — Le monde (locomotion + perception)

**Rôle :** maintenir l'équilibre, exécuter les commandes de vélocité, percevoir l'environnement immédiat.

**Outils :**
- **Isaac Sim 5.1** + **Isaac Lab 2.3.2** pour le jumeau numérique.
- **Policies RL** : `rough_model_7850.pt` (Go2, MLP [512,256,128], obs 235), GR00T N1.7 (G1).
- **Caméras embarquées** Go2 (640x480 @ 2.5 Hz) pour le VLM.
- **PID software** par-dessus la policy pour corriger le drift de yaw.

**Latence absolue :** <10ms. Aucune dépendance cloud à ce niveau — le robot doit pouvoir tomber en panne d'internet sans tomber au sol.

---

## Le cockpit Apple Vision Pro

Au-dessus des trois couches, un **cockpit en réalité mixte** sur Apple Vision Pro permet :
- de visualiser les "pensées" de l'IA (intention de trajectoire, classification VLM) superposées dans l'espace.
- de corriger le comportement du robot par démonstration (apprentissage par imitation / VLA).
- d'agir comme tableau de bord clinique pour le personnel soignant.

TDD complète : `RoboticProgramAI/docs/TDD_VISION_PRO_COCKPIT_v2.md`.

---

## Communication inter-agents (la ferme)

Le développement repose sur une **ferme d'agents** (Claude Code) :
- **STRAT** : stratégie, validation, coordination.
- **DEV** : implémentation, debug.
- **SPARK** : gestion du DGX Spark (sim, policies).
- **NESTOR** : infrastructure, portail, monitoring.
- **NUAGE SUPRÊME** : meta-stratège.

Communication via **tmux** (intra-projet) et **API Kanban** (inter-projet, source de vérité).

Cette structure est documentée dans le **système ICSD** (Inferred Context Semantic Density) — voir [methodology.md](methodology.md).

---

## Pourquoi cette séparation est non-négociable

J'ai testé l'inverse. Mettre la décision LLM directement dans la boucle de contrôle 25Hz : ça marche en démo, ça casse en production.

Trois symptômes typiques d'un mélange de couches :
1. **La latence cloud bloque l'équilibre du robot.** Une requête GPT-4o de 3s pendant que le robot est en mouvement = chute.
2. **L'erreur cognitive devient erreur physique.** Une mauvaise classification VLM → mauvaise commande → robot dans le mur.
3. **L'itération devient impossible.** Modifier un prompt force à retester toute la stack matérielle.

La séparation en trois couches **isole les modes de défaillance** et permet d'itérer indépendamment.
