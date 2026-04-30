# Résultats — Mesurés, reproductibles, datés

> Tout ce qui est listé ici est passé en validation. Les méthodes sont reproductibles. Les logs sont conservés.

---

## GR00T N1.7 sur Unitree G1 <a id="groot-n17"></a>

**Date :** avril 2026 (modèle publié fin mars 2026 par NVIDIA).

**Setup :** G1 (Education Pro) + DGX Spark, inférence GR00T N1.7 directement.

**Résultat :**
- Walk : OK, équilibre maintenu sur plus d'une minute.
- Turn : OK sur ±90°.
- Pas de chute spontanée sur les sessions de test (>5 min cumulées).

**Limites observées :**
- Fine-tuning par tâche reste indispensable pour des comportements spécifiques.
- Latence d'inférence acceptable mais variable selon la complexité de l'observation.

**Référence comparative :** WBC clone GEAR-SONIC (voir section suivante).

---

## WBC clone GEAR-SONIC <a id="wbc"></a>

**Date :** avril 2026.

**Objectif :** disposer d'un Whole-Body Controller bas niveau open-source comme baseline pour comparer GR00T.

**Résultat :** clone fonctionnel, intégré sur G1. Sert de point de référence pour mesurer ce que GR00T apporte vraiment vs un contrôleur classique. Conclusion partielle : GR00T excelle sur la **généralisation aux configurations inédites** (e.g. terrain incliné inconnu) — un WBC classique reste compétitif sur tâches strictement entraînées.

---

## Pipeline Hy3D → Isaac Sim <a id="hy3d"></a>

**Outil :** Tencent Hunyuan3D (Hy3D).

**Workflow :**
1. Prompt textuel ou image → génération mesh GLB par Hy3D.
2. SCP du GLB vers `/home/panda/robotics/assets/custom/` sur Spark.
3. `add_reference_to_stage()` dans le launch script Isaac Sim.
4. CollisionAPI appliquée si nécessaire (statique = convexHull).
5. Position et orientation configurables via CLI args.

**Assets livrés :**

| Asset | Taille | Usage |
|---|---|---|
| **BestFriend.glb** | 7.8 MB | Renard compagnon — mascotte des scénarios animaux |
| **Doctor.glb** | 9.2 MB | Humanoïde stylisé — cible des tests d'identification |
| **Go2 tron skin** | (custom) | Texture cyberpunk du quadrupède (style hybride Tron) |

**Coût marginal mesuré :** ~5 minutes pour un nouvel asset (vs ~3-5h pour modélisation manuelle Blender).

---

## Patrouille Go2 + VLM (E2E) <a id="patrol"></a>

**Scripts :** `patrol_circle.py`, `patrol_square.py`, `patrol_demo.py`, `patrol_mission1.py`, `patrol_mission2.py`, `patrol_mission3.py`.

**Pipeline complet validé :**

```
Caméra Go2 (640x480)
    ↓ JPEG @ 2.5 Hz
mono_robot_look.py
    ↓
GPT-4o VLM (vision)
    ↓ description texte
Filtre négation regex
    ↓ classification animal/humain
Décision agentique
    ↓ si humain :
    "IDENTIFIEZ-VOUS" → réponse simulée
    ↓
GPT-4o decision (vs authorized_personnel.json)
    ↓
AUTORISÉ ✅ ou ALARME 🚨
```

**Métrique : faux positifs.** Sans filtre de négation, le VLM produisait ~80% de faux positifs ("pas de personnes" matché comme "personnes"). Après filtre regex sur 4 patterns de négation FR/EN : **100% des faux positifs éliminés** sur 16 scans tests.

**Métrique : E2E.** Mission 3 (renard mur sud + Docteur David mur est) — le robot discrimine correctement animal/humain, identifie, et déclenche l'alarme appropriée. Validé en simulation.

---

## PID yaw correction <a id="pid"></a>

**Problème :** la policy `Velocity-Rough-Unitree-Go2` dérive d'environ 80° en 30 secondes en marche droite (`vx=0.3, vy=0, vz=0`). Limitation connue du modèle pré-entraîné.

**Solution :** PID software par-dessus la policy.

```python
YAW_KP = 0.03
YAW_KI = 0.003
YAW_KD = 0.003
YAW_WZ_MAX = 0.15
YAW_CORRECTION_HZ = 4
```

Le robot lit son yaw réel à 4 Hz, calcule l'erreur contre le cap désiré, injecte un wz correctif borné. La policy RL reçoit un wz non-nul ce qui l'aligne (et évite le crash de "rotation pure" qui apparaît avec `vx=0`).

**Résultat mesuré :**

| Métrique | Sans PID | Avec PID v2 |
|---|---|---|
| Drift latéral 30s | 5–8 m | **0.55 m** |
| Ratio drift/forward | 60–80% | **6.2%** |
| Yaw error stable | 80°+ | **±1°** |

Validé sur 9 m parcourus à 0.3 m/s, sur ground plane Isaac Sim.

---

## Compatibilité walk-these-ways / unitree_rl_gym

**Verdict :** incompatibles directement avec Isaac Sim 5.1 / Isaac Lab 2.3.

| Repo | Sim | Obs dims | Verdict |
|---|---|---|---|
| `Teddy-Liao/walk-these-ways-go2` | Isaac Gym Preview 4 (déprécié) | 70 + 2100 history | NON drop-in |
| `Improbable-AI/walk-these-ways` (MIT) | Isaac Gym Preview 4 | 42 (Go1 only) | NON applicable |
| `unitreerobotics/unitree_rl_gym` | Isaac Gym Preview 4 | 48 | NON drop-in |
| **Isaac Lab 2.3 (notre setup)** | Isaac Sim 5.1 | 235 (height scan) | — |

Pour porter une policy d'un de ces repos vers Isaac Lab : reconstruire le vecteur d'observations en respectant l'ordre et le scaling, fournir le buffer d'historique pour les architectures à adaptation module. Travail réaliste mais non trivial.

**Décision pratique :** rester sur le rough_model Isaac Lab natif + PID correction, fine-tuner si besoin spécifique.

---

## Benchmark VLM

**Image test :** caméra Isaac Sim 640x480 JPEG.

| Modèle | Latence | Throughput | Verdict |
|---|---|---|---|
| **Groq Llama-4-Scout 17B** | 0.34 s | 187 tok/s | Champion vitesse, utilisé en production |
| **GPT-4o** | 3.57 s | 7.6 tok/s | Plus précis, fallback cloud |
| **qwen3-vl Ollama (local)** | 9.15 s | 38 tok/s | Fallback offline |
| **Nemotron-12B-VL** | n/a | n/a | Bloqué (mamba-ssm pas aarch64) |

**Stratégie actuelle :** GPT-4o en principal pour la patrouille (qualité de raisonnement > latence), Groq pour les requêtes ultra-rapides où la précision est secondaire.

---

## Sprint 16, 17, 18 (cycle Sprint Robotics)

**Sprint 16 — Preuve d'évolution cognitive** (clos février 2026, 23/23 tâches).
Démonstration que la mémoire MonoCLI rend la navigation plus rapide et plus résiliente après chaque run. Phases A (baseline 38.2s), B (perturbation +39.5%), C (évoluée -6.7% distance, +17.5% résilience).

**Sprint 17 — Perturbation obstacle physique** (clos mars 2026, 7/7 tâches).
Obstacle 1.0×1.0×0.6m sur trajectoire, contournement cognitif vs aveugle. Cognitif +14% plus lent mais **prédictible et reproductible**. Rapport positionnement vs SayCan/RT-2/Helix/pi0/GR00T/UnifoLM.

**Sprint 18 — Patrouille adaptative** (clos mars 2026, 8/8 tâches).
Boucle génétique complète : percevoir → décider → agir → apprendre → évoluer. Gen 1 naïve 87.3s, Gen 2 évoluée 71.2s (-18.4%).

Documentation détaillée dans `RoboticProgramAI/docs/`.
