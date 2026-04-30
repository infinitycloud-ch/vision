# Roadmap — Ce qui vient, et ce qui reste à prouver

> Pas de promesse marketing. Des jalons techniques précis, datés quand c'est possible.

---

## Court terme (1-2 mois)

### Sim-to-real boustrophédon Go2 (Palier 1)
**Statut :** en cours.
- Bridge `go2_bridge.py` : endpoints `/move`, `/stop`, `/status` ✅ (avril 2026).
- BoustrophedonService Swift sur iPhone : en cours côté équipe MonoCLI.
- Test physique 10×10m attendu : mai 2026.
- Critère succès : 100/100 cellules couvertes, 0 chute, ARKit tracking maintenu.

### Mission patrouille avec discrimination animal/humain (physique)
**Statut :** validé en simulation, à porter sur Go2 réel.
- Pipeline VLM + identification déjà E2E en sim.
- Reste à valider en environnement réel avec assets 3D imprimés ou figurines.

### Capteur olfactif BME690
**Statut :** spec'd, hardware non commandé.
- Détection de gaz, recherche par signature olfactive.
- Use case : détection chute (urine) en EHPAD.
- ETA : courant T2 2026.

---

## Moyen terme (3-6 mois)

### G1 humanoïde — autonomie en environnement structuré
**Statut :** GR00T N1.7 inférence OK, autonomie 0.

Objectifs :
- Navigation point à point dans une chambre EHPAD simulée.
- Détection chute simulée et alerte.
- Dialogue naturel avec patient simulé.

### Apple Vision Pro cockpit — déploiement clinique
**Statut :** TDD v2.0 finalisé, implémentation pas démarrée.
- Visualisation des "pensées" du robot en MR.
- Apprentissage par démonstration / VLA via Vision Pro.
- ETA : T3 2026.

### Pipeline Hy3D enrichi
**Statut :** 3 GLB livrés, pipeline reproductible.

Objectifs :
- 20+ assets de scène EHPAD (lits, fauteuils, éclairages, signalétique).
- Génération automatique de scènes complètes (procédural + Hy3D).

---

## Long terme (6-12 mois)

### Pilote en EHPAD réel (Suisse)
**Statut :** discussions préliminaires HES-SO, Fegems.
- Critère go/no-go : succès des tests sim-to-real palier 1 + 2.
- Risque : régulation médicale, RGPD, sécurité.
- Stratégie : démarrage en mode observation passive (pas d'interaction physique avec patients).

### Validation Venture Kick / financement
**Statut :** pitch v1 rédigé.
- Compagnon CHF 5000 marché EHPAD Suisse.
- Application pour CHF 150k Phase 1.

### Open-sourcing partiel
**Statut :** réflexion en cours.

Décision à prendre : quelles briques rendre publiques ?
- **Probables open-source :** API contracts, templates Isaac Sim, pipelines de validation, mémoire ICSD.
- **Probable propriétaire :** moteur MonoCLI, méthodologie agentique complète, intégrations EHPAD.

---

## Limites assumées

Ce qui ne sera **pas** fait dans cette roadmap :

- **Entraînement de policies fondamentales from-scratch** — réservé aux acteurs avec cluster H100.
- **Hardware custom** — on reste sur Unitree + iPhone + Vision Pro + DGX Spark.
- **Internationalisation rapide** — focus Suisse (puis Europe francophone) avant tout.
- **Levée de fonds massive** — l'objectif est de démontrer qu'on peut avancer sans.

---

## Métriques de progression

À chaque sprint, deux questions :

1. **Qu'est-ce qui marche que je ne pouvais pas démontrer le sprint dernier ?**
2. **Qu'est-ce qui ne marche toujours pas, et pourquoi ?**

Pas d'OKR fancy. Pas de KPI à tableau de bord. Juste ces deux questions, à chaque revue.

C'est suffisant pour avancer vite. Et ça force l'honnêteté.
