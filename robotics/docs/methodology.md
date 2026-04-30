# Méthodologie — Simulation-first, ICSD, discipline d'itération

## Règle 1 — Rien ne touche le robot physique avant 100 runs en simulation

Une chute d'un G1 réel coûte ~8 000 CHF. Une chute en simulation coûte un `env.reset()`. Cette asymétrie est la base de toute la méthode.

**Conséquence pratique :** la qualité de la simulation est un produit en soi.

Demi-journée perdue cette session sur un mismatch de friction entre le ground plane Isaac et le `velocity_env_cfg` du training Unitree. Une autre demi-journée sur la compatibilité d'observations entre walk-these-ways et Isaac Lab 2.3. Ce ne sont pas des accidents — c'est le **prix réel** de la simulation-first. Et il vaut largement le prix d'un G1 cassé.

---

## Règle 2 — La stack reste mince

Pas d'orchestrateur Kubernetes. Pas de microservices. Pas de message broker. Trois scripts Python, un bridge UDP, un viewer Three.js. Quand quelque chose casse, je sais pourquoi en cinq minutes.

Comparaison : un projet équivalent vu chez un grand groupe utilisait Airflow + Kafka + Kubernetes + 4 services Python distincts. Au moindre bug, 30 minutes de log diving avant de localiser la couche fautive.

La complexité a un coût caché énorme pour un développeur indépendant. **Toute brique ajoutée doit prouver sa valeur.**

---

## Règle 3 — ICSD (Inferred Context Semantic Density)

C'est le système de communication développé pour la ferme d'agents (Claude Code).

**Principe :** pseudo-code dense > prose descriptive. 1 ligne = 1 concept. Compression > exhaustivité.

Exemple de mémoire ICSD :
```
policy = rough_model_7850.pt — MLP[512,256,128] — obs235 — act12
policy.probleme = yaw_drift_80deg_en_30s — rotation_pure(vx=0)=chute
policy.fix = PID_yaw(Kp=0.03,Ki=0.003,Kd=0.003) + arc_turn(vx=0.1+wz)
walk-these-ways = INCOMPATIBLE(Isaac_Gym_Preview4 ≠ Isaac_Sim_5.1)
```

Ces 4 lignes contiennent l'essentiel d'une session de 6h de debug. Un agent qui les relit après refresh reconstruit le contexte en quelques secondes.

**Contrepartie :** illisible pour quelqu'un qui n'a pas le contexte. C'est volontaire — c'est de la **mémoire technique** entre soi et soi-même (et entre agents pairs), pas de la documentation publique.

---

## Règle 4 — La preuve par l'image

Avant de déclarer un travail visuel terminé : **screenshot obligatoire**. Pas de "ça devrait marcher". Pas de "j'ai fait le diff". On voit, ou on n'a pas fini.

Pour la robotique : viewer Three.js custom (port 8080 sur Spark) qui montre en temps réel la position du Go2/G1, les assets, la grille mosaïque, les murs. Toute affirmation sur le comportement du robot s'accompagne d'un screenshot du viewer.

---

## Règle 5 — Honnêteté sur les blocages

Si quelque chose ne marche pas, le dire. Tout de suite. Au bon canal.

Liste honnête des points actuellement non résolus (au 30 avril 2026) :
- Sim-to-real pas franchi (tout est en simulation pour les missions cognitives complexes).
- Identification visuelle d'objets non-photoréalistes fragile (mesh Hy3D simplifié ≠ humanoïde reconnaissable par GPT-4o).
- Policy RL fatigue après ~50s en production (last_action accumule du drift).
- Pas de capteur olfactif intégré (BME690 prévu, pas encore implémenté).

Ces points sont **publics dans le repo et la roadmap**. Pas par naïveté — par stratégie. Un projet qui ne montre que ses succès est suspect. Un projet qui montre aussi ses limites est crédible.

---

## Règle 6 — Itération > planification longue

Pas de roadmap à 18 mois. Pas de Gantt à plusieurs niveaux. Des sprints de 1-2 semaines avec un livrable mesurable, et une revue immédiate.

Sprint 16 → 17 → 18 : trois sprints, trois preuves différentes, en un mois. Chacune débloque la suivante.

Cette discipline est typique des projets de recherche réussis. Elle nécessite par contre **un seul décideur** ou une équipe alignée. Avec 12 ingénieurs et un management intermédiaire, cette vitesse devient impossible.

---

## Règle 7 — La fin du prompt prédit la suite

À la fin de chaque interaction agent, **proposer une (et une seule) action logique suivante**. Pas un menu, pas une liste — une suggestion qui s'inscrit dans le flux du raisonnement.

C'est de la prédiction sémantique de la prochaine itération. Quand c'est bien fait, le décideur n'a qu'à dire "go" pour avancer. Quand c'est mal fait, on perd 5 minutes à clarifier.

---

## En résumé

| Règle | Effet mesurable |
|---|---|
| Simulation-first | 0 robot cassé en 6 mois |
| Stack mince | Debug moyen <5 min |
| ICSD | Reprise post-refresh en secondes |
| Preuve par image | 0 régression visuelle non détectée |
| Honnêteté blocages | Crédibilité technique vs vendeurs |
| Itération courte | 3 sprints majeurs / mois |
| Prédiction next | Décision en 1 mot vs 5 min |

Aucune de ces règles n'est nouvelle. Aucune n'est révolutionnaire. Mais **les appliquer toutes simultanément** sans compromis est inhabituel — et c'est ce qui produit la vélocité observée.
