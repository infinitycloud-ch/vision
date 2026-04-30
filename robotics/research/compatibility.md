# Compatibilité — État de l'écosystème policies Go2

## TL;DR

Aucun des trois repos open-source majeurs pour la marche du Go2 n'est directement compatible avec **Isaac Sim 5.1 / Isaac Lab 2.3** (notre stack).

Ils utilisent tous **Isaac Gym Preview 4** — déprécié par NVIDIA, remplacé par Isaac Lab. Les espaces d'observations diffèrent radicalement, ce qui empêche le drop-in d'une policy pré-entraînée.

---

## Tableau de compatibilité

| Repo | Robot | Sim requis | Obs dims | Action dims | Pré-trained | Compatible Isaac Sim 5.1 |
|---|---|---|---|---|---|---|
| **Teddy-Liao/walk-these-ways-go2** | Go2 | Isaac Gym Preview 4 | 70 + 2100 history | 12 | OUI (51 ckpts + JIT) | ❌ |
| **Improbable-AI/walk-these-ways** (MIT) | Go1 | Isaac Gym Preview 4 | 42 | 12 | OUI (Go1 only) | ❌ |
| **unitreerobotics/unitree_rl_gym** | Go2, G1, H1 | Isaac Gym Preview 4 | 48 (Go2) | 12 | NON pour Go2 | ❌ |
| **Notre setup (Isaac Lab 2.3)** | Go2 | Isaac Sim 5.1 | **235** (height scan) | 12 | rough_model_7850.pt | — |

---

## Détails par repo

### Teddy-Liao/walk-these-ways-go2

**Forces :**
- Pré-entraîné Go2 (51 checkpoints + JIT body + adaptation module).
- Supporte gait conditioning : trot, pronk, bound, pace.
- Architecture teacher-student avec adaptation module historique.

**Bloquant :**
- Observations : 70 dims par step + 2100 dims d'historique (30 steps × 70).
- Format complètement différent de notre obs 235 dims (qui inclut height scan 187).
- Décision pratique : non-portable directement.

**Effort estimé pour porter :**
- Reconstruire le vecteur d'obs en respectant l'ordre exact.
- Implémenter le buffer d'historique 30 steps.
- Charger les JITs avec le bon device map.
- 1-2 semaines de travail pour un développeur expérimenté Isaac Lab.

### Improbable-AI/walk-these-ways

Référence MIT, **Go1 uniquement**. Modèles non transférables au Go2 (URDF, dynamique, masses différentes). Utile uniquement comme implémentation de référence pour comprendre l'architecture.

### unitreerobotics/unitree_rl_gym

Repo officiel Unitree. Multi-robot (Go2, G1, H1, H1_2). Bonne qualité de code mais :
- Pas de policy pré-entraînée fournie pour Go2 (il faut entraîner soi-même).
- Obs 48 dims — incompatible obs 235 dims.

Décision : intéressant comme référence config (gains PD : Kp=20, Kd=0.5, decimation=4) mais pas drop-in.

---

## Notre choix

**Stratégie retenue :** rester sur le `rough_model_7850.pt` natif Isaac Lab + corrections software (PID yaw).

Raisons :
1. Compatibilité immédiate avec Isaac Sim 5.1 / Isaac Lab 2.3.
2. Le PID yaw correction résout 90% du problème de drift sans toucher la policy.
3. Le fine-tuning éventuel se fera dans le framework Isaac Lab natif si besoin.
4. Walk-these-ways apporte du gait conditioning intéressant mais **pas critique** pour notre use case (assistance, navigation lente sécurisée).

---

## Si vous voulez quand même porter walk-these-ways-go2

Voici les étapes minimales :

1. **Charger les JITs** :
   ```python
   body = torch.jit.load("checkpoints/body_latest.jit")
   adapt = torch.jit.load("checkpoints/adaptation_module_latest.jit")
   ```

2. **Reconstruire le vecteur obs** (70 dims, ordre crucial) :
   - projected_gravity (3)
   - commands (15) — vx, vy, yaw, height, frequency, gait[3], duration, footswing, pitch, roll, stance_width, stance_length
   - dof_pos (12)
   - dof_vel (12)
   - actions (12)
   - prev_actions (12)
   - clock_inputs (4)

3. **Maintenir le buffer d'historique** : ring buffer de 30 steps × 70 dims = 2100.

4. **Inférence** :
   ```python
   latent = adapt(obs_history)
   action = body(torch.cat((obs_history, latent), dim=-1))
   ```

5. **Valider PD gains et décimation** : Kp=25, Kd=0.6, decimation=4, action_scale=0.25.

Si tout est aligné, la policy devrait fonctionner. **Pas testé chez nous** — décision de ne pas investir le temps tant que le rough_model + PID suffit.
