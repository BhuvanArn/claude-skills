---
name: revue
description: >
  Revue pré-commit du diff courant pour un projet Unity — vérifie que c'est le BON changement,
  pas seulement un changement qui passe. Utiliser AVANT tout commit non trivial, quand
  l'utilisateur dit "revue", "review", "vérifie avant de committer", "est-ce le bon
  changement", ou demande si le travail est prêt à être commité/pushé. À utiliser aussi
  spontanément en fin de tâche, avant de proposer un commit — même si l'utilisateur ne le
  demande pas explicitement. Complète /code-review (chasse aux bugs) : ici on juge le
  périmètre, les contrats, la simplicité et l'alignement avec l'intention.
---

# Revue pré-commit — « est-ce le bon changement ? »

La CI prouve que les vérifications prévues passent. **Pas que nous avons vérifié les
bonnes choses.** Cette revue juge le changement lui-même, avant la lecture ligne par ligne.

## Étape 0 — Établir l'intention

Avant de juger le diff, reconstituer ce que le changement DEVAIT faire :

1. La demande d'origine (message utilisateur, jalon `ROADMAP.md`, critères d'acceptation).
2. Le contexte : quel jalon est en cours, quels garde-fous de scope du design doc
   (`docs/superpowers/specs/`) s'appliquent.

Sans intention claire, la revue ne peut pas juger l'alignement — la demander à
l'utilisateur plutôt que de la deviner.

## Étape 1 — Collecter le diff

```bash
git status --short
git diff HEAD          # ou git diff <base>..HEAD si des commits existent déjà
dotnet test Simulation.Tests   # état des tests AVANT de juger
```

## Étape 2 — Les 9 questions

Répondre à chacune **avec une preuve tirée du diff** (fichier:ligne), pas une impression.
Une question sans réponse vérifiable = point à creuser, pas à cocher.

| # | Question | Ce que ça veut dire ici |
|---|---|---|
| 1 | **Besoin** | Les critères d'acceptation sont-ils couverts ? Relire la demande, lister chaque critère, pointer où le diff le satisfait. |
| 2 | **Périmètre** | Pourquoi chaque fichier a-t-il été modifié ? Chaque fichier du diff doit se justifier par le besoin. Fichier inexpliqué = drapeau (scope creep ou artefact régénéré commité par erreur). |
| 3 | **Contrats** | Une API, un schéma ou un comportement existant a-t-il changé ? Ici : événements publics de la sim, format de sauvegarde, données ScriptableObject, seuils d'équilibrage. Changement de contrat = le signaler + `BREAKING CHANGE:` si une save devient incompatible. |
| 4 | **Cohérence** | Le diff respecte-t-il `docs/CONTRIBUTING.md`, `.editorconfig`, les ADRs ? En particulier : aucun `using UnityEngine;` dans le package sim, aucune règle de jeu dans `Assets/`. |
| 5 | **Tests** | Auraient-ils échoué avant le changement ? Vérifient-ils le comportement (pas l'implémentation) ? Un test qui passait déjà avant le fix ne prouve rien. |
| 6 | **Sécurité** | Entrées, permissions, secrets : pas de licence Unity, token ou credential dans le diff ; pas de chemin machine en dur ; prudence sur tout code qui lit des fichiers de save (entrée non fiable). |
| 7 | **Exploitation** | Erreurs, logs et rollback compréhensibles ? Côté Unity : la Console raconte-t-elle une histoire lisible ? Un état corrompu est-il récupérable (règle "reculer, pas détruire") ? |
| 8 | **Simplicité** | Existait-il une solution plus simple ? Chercher activement : moins d'abstraction, moins de fichiers, réutiliser l'existant. "YAGNI ruthless" est une décision de design du projet, pas une préférence. |
| 9 | **Alignement** | Le changement respecte-t-il l'intention GLOBALE, pas seulement les critères locaux de la tâche ? Relire les piliers du design et les garde-fous de scope : un changement peut cocher toutes les cases et quand même tirer le projet hors de sa trajectoire. |

## Étape 3 — Verdict

Rendre le résultat dans ce format exact :

```
## Revue : <résumé du changement en une ligne>

**Verdict : PRÊT À COMMITTER | CORRECTIONS NÉCESSAIRES | DISCUSSION REQUISE**

| Question | ✅/⚠️/❌ | Preuve / problème |
|---|---|---|
| 1. Besoin | ✅ | ... |
| ... | | |

### Bloquants
(ce qui empêche le commit — vide si aucun)

### Suggestions
(améliorations non bloquantes — à la discrétion de l'utilisateur)
```

Règles du verdict :

- ❌ sur les questions 1–5 ou 9 → **CORRECTIONS NÉCESSAIRES**. Corriger, puis re-passer la revue.
- Ambiguïté sur l'intention (question 9 indécidable) → **DISCUSSION REQUISE** : présenter
  les interprétations possibles à l'utilisateur au lieu d'en choisir une.
- Les suggestions ne bloquent jamais : les lister honnêtement, laisser l'utilisateur trancher.
- Ne pas gonfler la revue : si le diff est bon, dire qu'il est bon en une table et s'arrêter.
