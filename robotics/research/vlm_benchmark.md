# Benchmark VLM — Vision-Language Models pour patrouille robotique

## Setup

**Image test :** caméra Isaac Sim 640×480 JPEG (qualité 80, ~25 KB).
**Prompt :** "Décris ce que tu vois en 2 phrases. Y a-t-il une personne ou un animal ?"
**Date :** mars-avril 2026.
**Hardware :** appel cloud depuis Mac M3 Ultra ou Spark (10 Gbit Ethernet).

---

## Résultats

| Modèle | Latence p50 | Throughput | Coût (~) | Verdict |
|---|---|---|---|---|
| **Groq Llama-4-Scout 17B** | 0.34 s | 187 tok/s | <0.001 USD/req | Champion vitesse |
| **OpenAI GPT-4o** | 3.57 s | 7.6 tok/s | ~0.005 USD/req | Production actuelle |
| **Ollama qwen3-vl (local Spark)** | 9.15 s | 38 tok/s | 0 USD (local) | Fallback offline |
| **Nemotron-12B-VL** | n/a | n/a | n/a | Bloqué aarch64 |

---

## Détails par modèle

### Groq Llama-4-Scout 17B

**Le plus rapide.** 0.34s de latence end-to-end inclut le réseau. C'est plus rapide que ce qui est nécessaire pour la patrouille.

**Limites observées :**
- Cloudflare bloquait initialement les requêtes vision sans header User-Agent (erreur 1010). Fixé en ajoutant `User-Agent: patrol/1.0`.
- Description un peu plus générique que GPT-4o sur les scènes complexes.
- Les modèles vision préview Groq (`llama-3.2-11b-vision-preview`, `llama-3.2-90b-vision-preview`) ont été décommissionnés début avril 2026 — seul Llama-4-Scout supporte vision désormais sur Groq.

**Use case :** scans de routine quand la latence prime (chaque 5s pendant la patrouille). Ne plus utiliser pour les décisions critiques d'identification.

### OpenAI GPT-4o

**Production actuelle.** Migration depuis Groq décidée en avril 2026.

**Forces :**
- Qualité de raisonnement supérieure sur les scènes simulées (mesh non photo-réalistes).
- Capable de comprendre des prompts contextualisés (e.g. "Le Docteur dans cette simulation est représenté par une grande forme bleue").
- API stable, documentation complète, format multimodal standard.

**Faiblesses :**
- Latence 10× supérieure à Groq.
- Coût non négligeable à grande échelle.

**Use case actuel :** identification critique + décision d'autorisation d'accès. La latence est acceptée parce que la décision est ponctuelle (quelques fois par mission, pas en boucle).

### Ollama qwen3-vl (local sur Spark)

**Fallback offline.** Tourne entièrement sur le DGX Spark, pas de dépendance cloud.

**Forces :**
- 0 USD par requête, scalable.
- Pas de fuite de données (toutes les images restent sur Spark).
- Latence de 9s acceptable pour des analyses asynchrones.

**Faiblesses :**
- Latence 25× plus lente que Groq.
- Qualité légèrement inférieure à GPT-4o sur les nuances.

**Use case :** mode dégradé (panne internet) ou batch processing offline.

### Nemotron-12B-VL (NVIDIA)

**Bloqué.** Le modèle NVIDIA Nemotron Vision-Language requiert `mamba-ssm` qui n'a pas de wheel pour aarch64 au moment du test.

À surveiller — la situation peut évoluer si NVIDIA publie un wheel ARM officiel.

---

## Stratégie production actuelle

```
Décision critique (identification d'humain) → GPT-4o
   └─ acceptable car latence < 5s, ponctuel

Scan de routine (toutes les 5s) → GPT-4o (qualité prime ici aussi pour éviter les faux positifs)
   └─ pourrait basculer Groq si volume devient un facteur

Scan offline / batch → Ollama qwen3-vl local
   └─ aucun coût marginal, asynchrone
```

---

## Filtre de négation regex

**Problème observé sans filtre :** le VLM décrit "Il n'y a pas de personnes visibles" → le matcher naïf trouve le mot "personnes" → faux positif.

**Solution :** filtre regex sur 4 patterns de négation FR/EN appliqué après détection des mots-clés humains :

```python
NEGATION_PATTERNS = [
    r"(?:pas|no|not|aucun|sans|n.y a pas)\s+(?:de\s+)?(?:" + "|".join(HUMAN_KEYWORDS) + r")",
    r"(?:" + "|".join(HUMAN_KEYWORDS) + r")\s+(?:not|aren.t|isn.t|n.est pas)",
    r"(?:no|zero|aucun)\s+\w*\s*(?:" + "|".join(HUMAN_KEYWORDS) + r")",
    r"n.y a\s+(?:" + "|".join(HUMAN_KEYWORDS) + r")",
]
```

**Métrique :** sur 16 scans tests post-filtre, **100% des faux positifs éliminés**. Aucun faux négatif observé (tous les humains réels ont été détectés).

---

## Leçons apprises

1. **Toujours envoyer un User-Agent** sur les API VLM cloud. Cloudflare bloque sinon.
2. **La latence VLM impose un design asynchrone.** Le robot ne peut pas attendre 3s pour décider — soit on stoppe pendant le scan, soit on fait du pipeline parallèle.
3. **Les modèles 3D non photo-réalistes piègent les VLM.** Une sphère bleue représentant un docteur n'est pas reconnue. Solutions : meilleurs assets (Hy3D), ou prompts adaptés contextuellement.
4. **Le filtre de négation est non-négociable.** Sans lui, ~80% de faux positifs. Avec : 0%.
