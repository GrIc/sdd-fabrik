# Protocole AUDIT — le gardien : détecter la dérive, régénérer l'index

> Rituel **hebdomadaire / pré-release / à la demande**. AUDIT est le gardien du
> couplage spec↔code. Il détecte la dérive de façon **mécanique** (git diff, pas
> relecture), régénère `SOURCE.md`, écrit un rapport daté, et signale les
> captures non triées.
>
> En topologie détachée, le contrat de commit même-commit est impossible :
> **AUDIT est la défense principale** et se joue à rythme soutenu (idéalement à
> chaque session de compilation côté repo code, au minimum hebdomadaire).
>
> Exécutable par tout agent sachant lire du markdown et lancer `git`.

---

## Étapes

### 1. Détection de dérive — MÉCANIQUE, module par module

Pour **chaque** spec `source/modules/<slug>.md`, lire son frontmatter
(`sync-repo`, `last-sync`, `governs`) et lancer le diff mécanique :

```bash
git -C <sync-repo> diff --stat <last-sync>..HEAD -- <governs...>
```

- `<sync-repo>` = valeur du champ `sync-repo` (`.` en monorepo, sinon le chemin
  du clone local du repo code, p. ex. `../code-repo`).
- `<last-sync>` = SHA enregistré au dernier point de sync.
- `<governs...>` = tous les chemins listés dans `governs`.

**Règle de verdict.** Le module passe `status: drifted` **si et seulement si** :

> le diff est **non vide** (le code a bougé sous la spec depuis `last-sync`)
> **ET** la spec elle-même n'a pas été mise à jour depuis `last-sync`.

Ce mécanisme est **déterministe** : deux agents sur le même état
(`last-sync`, `HEAD`, `governs`) produisent exactement le **même verdict**. La
dérive n'est **jamais** jugée par relecture humaine.

Script de détection complet, à rejouer tel quel (à lancer depuis la racine du
repo source). Il gère les deux écritures YAML de `governs` : liste inline
`[a, b]` **et** liste en bloc (`governs:` suivi de lignes `  - chemin`) :

```bash
#!/usr/bin/env bash
# Audit de dérive — détection mécanique par module.
set -euo pipefail
for spec in source/modules/*.md; do
  [ -e "$spec" ] || continue
  module=$(sed -n 's/^module:[[:space:]]*//p'    "$spec" | head -1)
  sync_repo=$(sed -n 's/^sync-repo:[[:space:]]*//p' "$spec" | head -1)
  last_sync=$(sed -n 's/^last-sync:[[:space:]]*//p'  "$spec" | head -1)
  # governs inline : governs: [a, b]
  governs=$(sed -n 's/^governs:[[:space:]]*\[\(.*\)\]/\1/p' "$spec" | head -1 \
            | tr ',' '\n' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
  # governs en bloc : lignes "  - chemin" entre "governs:" et le champ suivant
  if [ -z "$governs" ]; then
    governs=$(awk '/^governs:[[:space:]]*$/{f=1;next} f&&/^[[:space:]]*-[[:space:]]/{sub(/^[[:space:]]*-[[:space:]]*/,"");print;next} f&&/^[^[:space:]-]/{f=0}' "$spec")
  fi
  [ -z "${last_sync:-}" ] && { echo "SKIP  $module (pas de last-sync)"; continue; }
  diff=$(git -C "$sync_repo" diff --stat "$last_sync"..HEAD -- $governs || true)
  if [ -n "$diff" ]; then
    echo "DRIFT $module  (code bougé depuis $last_sync)"
    echo "$diff" | sed 's/^/        /'
  else
    echo "OK    $module"
  fi
done
```

Pour chaque module `DRIFT` dont la spec n'a pas été mise à jour depuis
`last-sync`, éditer son frontmatter → `status: drifted` et l'inscrire dans le
rapport (étape 3).

### 2. Régénérer `SOURCE.md`

`SOURCE.md` est **généré, jamais édité à la main** (sinon « la carte qui ment »).
Le reconstruire entièrement à partir du frontmatter des modules + de l'état git :

- **Table des modules** : une ligne par `source/modules/*.md` avec `module`,
  `status`, `governs`, `last-sync`, `sync-repo`, et la **santé de sync** (OK /
  DRIFT / SKIP issue de l'étape 1).
- **Liste des issues** : `source/issues/features/*` et `source/issues/bugs/*`
  (nom + une ligne de résumé + statut si présent en frontmatter).
- **File INTAKE** : les stubs `status: captured` (voir étape 4).

Écraser `source/SOURCE.md` avec le contenu régénéré.

### 3. Écrire un rapport daté dans `reports/`

Créer `source/reports/audit-<YYYY-MM-DD>.md` contenant :

- La date et le `HEAD` du repo code audité (`git -C <sync-repo> rev-parse HEAD`).
- Le verdict par module (OK / drifted), avec le `--stat` pour les modules
  dérivés.
- Les captures `captured` vieillissantes (étape 4).
- Les actions de réconciliation prises ou recommandées (étape 5).

> Ne jamais écraser un rapport antérieur : les rapports sont un **historique
> daté**, append-only par nature (un fichier par passe).

### 4. Signaler les captures non triées

Lister les stubs `status: captured` (issus du réflexe INTAKE) et repérer ceux
qui **vieillissent** (file d'attente qui stagne). Les inscrire dans le rapport
comme point de tri. Un fix commité **sans** issue correspondante est aussi de la
dérive — le signaler.

### 5. Réconciliation — trancher les divergences

Quand code et spec divergent, appliquer la règle :

> ## LE CODE GAGNE SUR LES FAITS, L'INTENT GAGNE SUR LE BUT.

- **Le code a divergé pour de bonnes raisons** (le monde réel a corrigé un
  *fait* que la spec décrivait mal — un endpoint qui n'existe pas, une API qui a
  changé) → **backfiller la spec** pour qu'elle dise la vérité du code, puis
  remettre `last-sync = HEAD` et repasser le module en `compiled`. *Le code
  gagne sur les faits.*
- **Le code a trahi l'intention** (il fait autre chose que ce que la vision
  voulait) → **rouvrir une issue** ; c'est le **code** qui doit revenir dans le
  rang, pas l'intention qu'on réécrit. Le module reste `drifted` jusqu'à ce que
  COMPILE le corrige. *L'intent gagne sur le but.*

Le discernement porte donc sur la **nature** de la divergence : un désaccord de
*fait* (le code sait mieux) vs un désaccord de *but* (l'intention sait mieux).

---

## Sortie attendue

- Modules dérivés passés en `status: drifted`.
- `source/SOURCE.md` régénéré sans intervention manuelle.
- `source/reports/audit-<date>.md` écrit (verdicts + captures + réconciliations).
- Décisions de réconciliation appliquées (backfill spec ou réouverture d'issue).

---

## Enchaînement

`AUDIT` détecte → `COMPILE` corrige les `drifted` qui ont trahi l'intent →
`AUDIT` re-vérifie. Boucle de gardiennage.
