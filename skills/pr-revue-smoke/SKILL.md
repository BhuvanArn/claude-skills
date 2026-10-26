---
name: pr-revue-smoke
description: >
  Méthode pour toute nouvelle PR, dans n'importe quel dépôt : la PR s'ouvre en brouillon
  (draft), deux sous-agents la relisent (la forme, puis le code), un troisième fait le
  smoke, et seulement ensuite elle passe « ready for review ». À utiliser dès qu'on ouvre
  ou crée une PR / pull request (`gh pr create`), et quand l'utilisateur demande de passer
  une PR en ready (« prête », « ready for review »), de « lancer les revues » ou « le smoke »
  d'une PR. Suppose GitHub et `gh`.
---

# PR : brouillon, deux revues, un smoke, puis ready

Méthode personnelle, pas une règle du dépôt : elle ne s'écrit ni dans un `AGENTS.md` ni
dans un `CLAUDE.md` versionné. Elle s'ajoute aux règles du dépôt, elle ne les remplace
pas (procédure de PR, commande de vérification, droits, captures, modèle de description).

Prérequis : dépôt GitHub et `gh` authentifié. Si le dépôt n'accepte pas les brouillons
(dépôt privé sur une offre gratuite), ouvrir la PR avec « [WIP] » en tête du titre et
retirer le préfixe à l'étape 5.

## 0. Repérer ce que le dépôt fournit déjà

Avant d'ouvrir quoi que ce soit, lire `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, le
modèle de PR (`.github/pull_request_template.md` ou `.github/PULL_REQUEST_TEMPLATE/`) et
`.claude/skills/` du dépôt, et noter :

- **la procédure de PR** du dépôt (modèle de description, branche cible, captures) ;
- **la commande de vérification** (`make check`, `npm test`, `pnpm verify`… ce que le
  dépôt utilise, pas une supposée) ;
- **les skills de revue et de smoke** propres au dépôt, qui priment sur les axes
  génériques ci-dessous ;
- **le public** de la description : si un relecteur ou décideur n'est pas technique,
  la description s'ouvre par « **En bref.** » en mots simples, garde les sections du
  modèle, ajoute « Ce qui reste à décider » s'il y a lieu, et plie le détail technique
  dans un `<details>`.

## 1. Ouvrir en brouillon

La procédure du dépôt s'applique telle quelle, avec une seule différence :
`gh pr create --draft`. Une PR déjà ouverte se remet en brouillon avec
`gh pr ready <N> --undo` avant les passes.

Diff de plus de ~400 lignes hors fichiers générés : proposer de le découper avant les
passes, une revue sur un gros diff survole. PR empilée : `--base <branche parente>`,
chaque PR relue sur son seul diff, et passée ready de la plus basse à la plus haute.

## 2. Deux sous-agents de revue

Lancés ensemble, dans le même message, `subagent_type: general-purpose`, en lecture
seule (ils ne modifient, ne commitent ni ne poussent rien). Avant de les lancer, se placer
sur la branche de la PR (`gh pr checkout <N>`) : ils lisent l'arbre de travail. Chacun
reçoit : le numéro et la branche, la commande `gh pr diff <N>` (fonctionne aussi depuis
un fork), la description de la PR et le besoin d'origine (issue liée, demande de
l'utilisateur), l'axe ci-dessous, et la consigne :

> Vérifier chaque constat dans le code avant de l'écrire. Chaque constat avec
> `fichier:ligne`, le défaut, pourquoi il compte, la correction, et l'étiquette
> `MECHANICAL` (correction évidente, sans choix) ou `DECISION` (plusieurs options
> défendables, ou un choix produit). Finir par un paragraphe « vérifié et sain » qui dit
> ce qui a été couvert.

Leur donner le diff et l'axe, **pas ses propres conclusions** : un relecteur à qui l'on
souffle l'hypothèse la confirme.

| Sous-agent | Axe |
|---|---|
| 1 : la forme | Bonne approche pour le besoin ? La plus simple possible ? Réutilise l'existant (recherche dans le code) plutôt que de dupliquer ? Suit les précédents et les règles d'architecture du dépôt ? Périmètre : rien hors sujet dans le diff ? La doc et les notes de décision voyagent avec le code ? Si le diff touche aux droits, aux migrations, aux secrets ou à l'authentification : skill de sécurité du dépôt, ou `security-review`, en plus. |
| 2 : le code | Skill de revue de code du dépôt s'il existe, sinon `code-review high <N>` (sans `--fix` ni `--comment`), constats reformulés en `MECHANICAL`/`DECISION` : justesse, cas d'erreur et limites, tests (ce qu'ils prouvent vraiment, comportement resté sans test), commentaires vrais et qui disent pourquoi, code mort, secrets ou données personnelles dans les journaux. |

Si la revue de forme change la forme du code, relancer la revue de code après la
correction : sinon elle a poli ce qui a été réécrit.

## 3. Trier et corriger

- Revérifier soi-même chaque constat dans le code avant d'agir. Un faux constat ne
  s'applique pas ; le dire.
- `MECHANICAL` : corriger, avec le test qui l'aurait attrapé quand c'est possible.
- `DECISION` : présenter à l'utilisateur, options et recommandation, et attendre. Une
  décision produit se range aussi dans « Ce qui reste à décider » de la description.
- Commande de vérification du dépôt une fois tout corrigé, puis **un seul** push.
- Un commentaire court par revue sur la PR : ce qui a été couvert, corrigé (commits),
  laissé en décision. Dans le registre du public de la PR.

## 4. Un sous-agent de smoke

Après les corrections, sur la dernière tête de branche, un sous-agent `general-purpose`.
Un seul smoke à la fois sur la machine : ports et mémoire.

- Skill de smoke du dépôt s'il existe, sinon skill `run`.
- Jouer **le parcours que le diff touche**, pas un parcours au hasard. Interface : à la
  taille d'écran de l'usage réel (téléphone si c'est le cas). Changement sans écran :
  se prouve par son effet (ligne de journal, tâche traitée, endpoint qui répond, sortie
  de la CLI).
- Diff qui touche au build, aux dépendances, aux variables d'environnement ou à la
  configuration d'exécution : smoke sur un build de production ou un déploiement de
  prévisualisation, pas seulement le serveur de dev.
- Captures si le changement est visible, selon la procédure du dépôt.
- Arrêter ce qui a été lancé (serveurs, conteneurs, ports), **toujours, même en échec**.
- Le sous-agent ne corrige rien : un défaut se rapporte. Smoke en échec : retour à
  l'étape 3, puis nouveau smoke sur la nouvelle tête.

Le sous-agent ne poste rien d'extérieur sans accord (messageries, hébergement d'images,
commentaires) : il capture, écrit son rapport et s'arrête. PR de documentation seule :
smoke sans objet, le dire dans le commentaire au lieu de lancer un sous-agent.

Commentaire de smoke sur la PR : ce qui a été joué, sur quoi (local, données factices,
préproduction), le résultat, et ce qui n'est **pas** couvert.

## 5. Passer ready

Seulement quand tout est vrai, chacun vérifié à l'instant (pas de mémoire) :

- deux commentaires de revue et un de smoke sur la PR ;
- aucun `MECHANICAL` ouvert ; chaque `DECISION` tranchée par l'utilisateur ou écrite
  dans la description ;
- CI verte sur le dernier commit (`gh pr checks <N> --watch` attend la fin des contrôles
  en cours). Sans CI : la commande de vérification du dépôt, lancée sur ce commit, en
  tient lieu, et le commentaire de smoke le dit ;
- pas de conflit avec la base (`gh pr view <N> --json mergeable`) ;
- aucun commit poussé après le smoke ; sinon refaire le smoke (et la revue de code si le
  code a changé). Un rebase sans conflit ne demande que la CI ; un conflit résolu à la
  main demande les deux.

Puis `gh pr ready <N>`. Ne pas fusionner : la fusion suit la règle du dépôt.

## Pour l'activer partout

Installer le skill dans `~/.claude/skills/`, pas dans le `.claude/skills/` d'un dépôt :
versionné là, il deviendrait une règle imposée aux autres contributeurs. Puis ajouter une
ligne au `~/.claude/CLAUDE.md` personnel :

```markdown
- **Toute nouvelle PR s'ouvre en brouillon (draft)**, puis deux sous-agents la relisent
  et un troisième fait le smoke, avant `gh pr ready`. Procédure : skill `pr-revue-smoke`.
```
