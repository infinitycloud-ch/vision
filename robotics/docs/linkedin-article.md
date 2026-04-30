# Bâtir une infrastructure de robotique cognitive depuis la Suisse — seul, avec un GB10 Blackwell sous le bureau

*Minh-Tam Dang — Infinity Cloud Sàrl*

---

## Un constat qui dérange

La robotique sérieuse — celle qui sort des démos LinkedIn et finit sur le terrain — coûte cher. Très cher. Une équipe de douze ingénieurs, un cluster cloud à six chiffres par mois, dix-huit mois avant le premier prototype intéressant. C'est le pattern dominant. Je le constate, je ne le juge pas : il a produit la majorité des résultats publiés en 2024–2026.

Mais ce n'est pas la seule voie.

Depuis fin 2025, je construis seul, depuis Lausanne, une infrastructure complète de robotique cognitive. Quadrupède Unitree Go2, humanoïde G1, jumeau numérique Isaac Sim, pipeline génération 3D, navigation autonome, identification VLM, mémoire persistante. Tout fonctionne sur **un NVIDIA DGX Spark — puce GB10 Blackwell, 100W de TDP, posé sur un bureau**, et un Mac M3 Ultra qui fait office de hub.

Ce post n'est pas une plainte sur les budgets, ni une promesse marketing. C'est un retour d'expérience technique sur ce qui marche, ce qui casse, et ce qu'on apprend quand on est obligé d'aller à l'essentiel.

---

## Le pari matériel : 100W versus mégawatts

Un datacenter d'entraînement RL classique consomme l'équivalent énergétique d'une commune de 5 000 habitants. Le DGX Spark consomme moins qu'un sèche-cheveux. La différence n'est pas qu'idéologique : elle change ce qu'on peut **se permettre** d'itérer.

- 128 GB de mémoire unifiée (CPU/GPU partagée) — assez pour Isaac Sim 5.1, Isaac Lab 2.3, et plusieurs policies en parallèle.
- Architecture aarch64 (ARM) — l'écosystème ML aarch64 a mûri, mais réserve quelques surprises (PyTorch CUDA disponible, certains binaires Python ML pas encore).
- Refroidissement passif suffisant pour des sessions de plusieurs heures sans throttling.

Pour un développeur indépendant, c'est l'équivalent d'avoir un labo NVIDIA sous la main. Pas un substitut au DGX H100 d'un grand groupe — un **outil suffisant pour boucler un cycle complet d'apprentissage** en simulation, déployer sur le robot physique, mesurer, recommencer.

---

## Ce qui tourne aujourd'hui

Quelques briques validées sur banc, sans triche, et que je peux démontrer.

**1. Inférence GR00T N1.7 sur Unitree G1**
NVIDIA a publié GR00T N1.7 fin mars 2026. Le modèle tourne en inférence sur le DGX Spark. Le G1 marche, tourne, conserve son équilibre. J'ai aussi cloné le Whole-Body Controller du projet GEAR-SONIC pour avoir un point de comparaison sur les bas niveaux. Conclusion partielle : N1.7 est exploitable hors-laboratoire, mais le fine-tuning par tâche reste indispensable.

**2. Pipeline Hy3D pour assets de scène**
Tencent Hunyuan3D génère des meshes GLB exploitables directement dans Isaac Sim via `add_reference_to_stage`. Trois assets livrés et intégrés dans la scène hôpital : *BestFriend* (renard compagnon, 7,8 MB), *Doctor* (humanoïde stylisé, 9,2 MB), *Go2 tron skin* (texture custom du quadrupède). Le coût marginal d'un nouvel objet de scène est passé de plusieurs heures (modélisation manuelle) à quelques minutes.

**3. Patrouille autonome avec discrimination animal/humain**
Le Go2 longe les murs d'une chambre simulée, scanne via la caméra embarquée toutes les 5 secondes, envoie l'image à GPT-4o avec un prompt générique ("y a-t-il un animal ou une personne ?"). Animal détecté → log + on continue. Humain détecté → arrêt + protocole d'identification → vérification contre une liste autorisée → décision agentique. Pipeline complet validé end-to-end : caméra → VLM → classification → décision → action.

**4. Correction PID en temps réel sur la dérive RL**
Le modèle de marche actuel dérive d'environ 80° en 30 secondes (limitation connue de la policy *Velocity-Rough-Unitree-Go2*). Un PID software lit le yaw réel à 4 Hz, calcule l'erreur contre le cap désiré, injecte un wz correctif borné. Résultat mesuré : dérive latérale ramenée de ~70 % à 6 % de la distance parcourue. Pas d'entraînement, pas de fine-tuning — juste un retour d'asservissement classique appliqué au-dessus de la policy.

---

## La méthode : simulation-first, discipline d'itération

Une seule règle : **rien ne touche le robot physique avant d'avoir tourné cent fois en simulation**. Ce n'est pas dogmatique, c'est économique. Une chute d'un G1 réel coûte 8 000 CHF. Une chute d'un G1 simulé coûte un `env.reset()`.

Le corollaire — moins évident — c'est que **la qualité de la simulation est un produit en soi**. Demi-journée perdue cette semaine sur un mismatch de friction entre le ground plane Isaac et le velocity_env_cfg du training (Unitree). Une demi-journée de plus en compatibilité d'observations entre les policies du milieu open-source (walk-these-ways, unitree_rl_gym, GR00T) et l'Isaac Lab 2.3 actuel : **les trois utilisent Isaac Gym Preview 4 (déprécié) et leurs spaces d'observations sont incompatibles**. Ce n'est pas un drame, c'est une donnée.

Itérer vite suppose aussi de **garder la stack mince**. Pas d'orchestrateur Kubernetes. Pas de microservices. Trois scripts Python, un bridge UDP, un viewer Three.js. Quand quelque chose casse, je sais pourquoi en cinq minutes.

---

## Ce qui ne marche pas (encore)

Par honnêteté, parce qu'un post technique sans points négatifs est toujours suspect :

- **L'identification visuelle d'objets non-photoréalistes est fragile.** Un mesh Hunyuan3D simplifié n'est pas reconnu comme humanoïde par GPT-4o. Solution actuelle : prompts adaptés au contenu de la scène. Solution propre : meilleurs assets ou modèles VLM fine-tunés.
- **La policy RL fatigue après ~50 secondes en production.** Le `last_action` accumule des erreurs hors épisodes. Reset périodique = workaround. Retraining ciblé = vraie réponse.
- **Le sim-to-real n'est pas franchi.** Tout ce que je décris ici tourne en simulation et sur le quadrupède en environnement contrôlé. La traversée vers l'autonomie en environnement humain réel — c'est l'objet des prochains mois.

---

## Où ça va

L'objectif d'Infinity Cloud n'est pas la performance pour la performance. C'est de mettre une présence robotique fiable au service des populations fragiles : maintien à domicile, soutien cognitif, sécurité passive en EHPAD. Pas de timeline marketing, pas d'annonces avant validation clinique. Mais un pipeline qui tourne, des résultats reproductibles, et une infrastructure qui ne dépend ni d'un cluster cloud, ni d'une équipe de douze ingénieurs.

Je publierai d'autres retours techniques dans les semaines qui viennent. Si vous travaillez sur des problématiques voisines — robotique d'assistance, VLA, simulation-first — la conversation m'intéresse.

---

#Robotics #NVIDIA #Blackwell #Isaac #IsaacSim #Switzerland #IndieAI #UnitreeG1 #UnitreeGo2 #SimulationFirst #VLM #ROS2 #DGXSpark #GB10 #Hy3D #GR00T
