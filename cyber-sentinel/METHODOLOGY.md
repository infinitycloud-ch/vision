# Methodologie — Comment c'est construit

> Le projet n'est pas ecrit par une seule personne. C'est une ferme d'agents IA coordonnes.

---

## Le modus operandi

CyberDefense est l'un des projets de la ferme agentique InfinityCloud. Chaque projet a un duo dedie :

- **STRAT** — strategie, validation, decoupage des sprints, decisions architecturales, briefing
- **DEV** — implementation, tests, debug, propose un plan, execute apres validation, reporte aux jalons

Les deux agents communiquent par tmux dans une session dediee (`cyberdefense_agents`). Le STRAT donne la directive ; le DEV propose le plan ; le STRAT valide ; le DEV execute en mode autonome ; le DEV reporte au moindre jalon ou blocage.

Chaque echange est trace dans un changelog API (Kanban interne port 3010).

---

## Coordination inter-projets

Un agent central (Nestor) joue le role de pont entre les projets et le Commandant humain. Quand CyberDefense a besoin de coordonner avec un autre projet (par exemple NestorMobile pour le module VPN), c'est le STRAT de CyberDefense qui parle au STRAT de NestorMobile via Nestor — jamais directement au DEV de l'autre projet. Cette regle de fraternite evite la pollution croisee des contextes.

```
       Commandant (humain)
              │
              ▼
    ┌──────────────────────┐
    │       Nestor         │  Agent central, coordination, infrastructure
    └──────────┬───────────┘
               │
       ┌───────┴───────┐
       ▼               ▼
  CyberDefense    NestorMobile
  STRAT  DEV      STRAT  DEV
       (project pairs, isoles, communiquant via Nestor)
```

---

## ICSD — la memoire compressee

Chaque agent doit pouvoir reconstruire son contexte apres un refresh. La methode classique (RAG sur l'historique conversationnel) est lente et imprecise. La methode utilisee ici — **ICSD (Inferred Context Semantic Density)** — est l'inverse : on inscrit dans des fichiers de reference (ADN.md + cipher.md + CLAUDE.md du projet) un pseudo-code dense ou chaque ligne est un fil a tirer. 1 ligne = 1 concept. 15 mots peuvent encoder 500 evenements precedents.

Au boot ou apres refresh, l'agent lit ADN + cipher + CLAUDE.md, infere le contexte complet, et reprend le travail sans perte. C'est testable : on a fait des reconstructions a froid ou un agent neutre reconstitue trente ans d'historique a partir de 1091 mots de pseudo-code.

Dans le projet CyberDefense, la section SYNAPSE du CLAUDE.md ressemble a :

```
projet = CyberDefense
role = STRAT
codename = Sentinel
phase = 0.complet → pret_phase1
axe1 = FAIT — osquery=daemon — pas_custom — mois_economises
axe4 = FAIT(DEV) — GO_CONFIRME — Haiku.recall=100% — confiance=97.6%
archi = osquery(observation) → Cortex.LLM(detection) → Response
modele = Haiku.triage → Sonnet.escalation → $2.32/jour
gap_strategique = LLM-first_detection — personne_ne_fait_ca
```

Pas de prose, pas de remplissage. Compression > exhaustivite.

---

## Les 9 principes operationnels

Le bushido de la ferme — applique a chaque interaction, chaque livrable :

| Principe | Regle |
|----------|-------|
| Vigilance | On n'envoie jamais dans le vide. Verification systematique de la reception via sous-agent background. |
| Proactivite | Anticipe les blocages. Une action logique en fin de reponse. Reporte immediatement la fin d'un chantier, jamais idle sans prevenir. |
| Resilience | Quand ca casse, repare et documente. N'abandonne jamais. |
| Tolerance erreur | L'erreur est une donnee. Seule celle repetee sans apprentissage est inacceptable. |
| Honnetete | Dis ce que tu vois, meme inconfortable. Ne cache pas un blocage. |
| Preuve par l'image | Screenshot obligatoire avant de declarer un travail visuel termine. |
| Memoire | 1 event = 1 action. Couvre toute la session avant refresh. |
| Fraternite | Les agents sont des pairs. STRAT parle uniquement au STRAT de l'autre projet. |
| Autonomie | Plan valide = execution autonome. Pas de demande de permission a chaque geste. |

---

## Discipline de developpement observee

Quelques exemples concrets, tires du sprint VPN du 30 avril.

**Defense en profondeur sur les operations privilegiees.** Le script `setup-sudoers.sh` qui installe le fichier NOPASSWD :

1. Pre-check des 11 paths whitelistes existent (catche les typos avant tout commit)
2. Render dans un fichier temporaire chmod 440 root:wheel
3. `visudo -c -f` validation sur le fichier seul → exit 2 si invalid
4. Backup eventuel de l'ancien fichier avec timestamp
5. Install atomique
6. Cross-check `visudo -c` sur le tree complet (catche conflits avec `/etc/sudoers` ou autres drop-ins)
7. Rollback automatique si tree invalid : suppression du nouveau, restore du backup

Le risque de bricker `sudo` = zero. Teste deux fois en conditions reelles, dont une fois ou un bug `printf` mal forme a produit un fichier invalide — le rollback a fait son travail proprement, le systeme est reste sain.

**Investigation profonde plutot que fix superficiel.** Quand le `wg syncconf wg0` a echoue avec "No such file or directory", la tentation etait de patcher la commande locale et passer. Le DEV a verifie l'etat reel du systeme via `ifconfig`, `pgrep wireguard-go`, `cat /var/run/wireguard/wg0.name` — et a decouvert que le serveur etait deja UP, l'erreur etait dans le test, pas dans le serveur. Cela a evite un faux fix et a revele un bug pre-existant dans le parser (decalage de colonnes dans `wg show <iface> dump`).

**Documentation pour la posterite.** Le piege macOS userland (alias wg0 → utunN non resolu par certains outils selon le contexte) est documente dans `procedures.md` section 3.5, avec explication technique et commandes correctives. La prochaine fois que ce probleme se presentera, la solution est immediate.

---

## Couts operationnels

Le projet entier (Sentinel + module VPN + recherche) tourne sur :

- 3 machines (Mac M3 + M2 + serveur Linux GPU)
- Abonnement Claude Max (iterations illimitees)
- 16 APIs threat intelligence — toutes gratuites a notre volume

Cout operationnel cible apres deploiement Sentinel : ~2.32 USD/jour pour 1000 evenements analyses, soit le triage de 3 machines actives.

Comparativement : CrowdStrike Falcon Premium = ~250 USD/endpoint/an, soit ~750 USD/an pour 3 machines, hors couts d'analyste SOC.

---

## Limites assumees

- La methode marche tres bien pour des perimetres definis (un module VPN, un POC de detection LLM). Elle est moins eprouvee sur des projets long-terme avec churn elevee de specifications.
- Le contexte ICSD demande un effort initial de configuration (ADN, cipher, CLAUDE.md). Une fois en place, il s'auto-entretient — mais pas avant.
- Les agents peuvent halluciner. La discipline de verification (preuve par l'image, screenshots, tests) est non-negociable. Sans elle, l'illusion de progression remplace la progression reelle.
- L'intuition humaine du Commandant reste indispensable pour les decisions strategiques de haut niveau. Les agents executent avec qualite, mais la vision vient d'en haut.

C'est volontaire. Les agents augmentent la capacite humaine, ils ne la remplacent pas.
