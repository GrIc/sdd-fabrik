# Protocole GENESIS — « aide-moi à créer un produit »

> Point d'entrée **greenfield** : aucun code n'existe encore, seule une idée.
> GENESIS transforme une intention floue en un socle `intent/` + des premières
> specs `modules/` en `status: draft`, puis scaffolde le repo code et son
> amorçage agent.
>
> Exécutable par tout agent (Claude Code, Antigravity…) sachant lire du markdown
> et lancer `git`. Aucune dépendance outil.

---

## Quand jouer GENESIS

- Le produit n'a pas encore de code, ou le code est un prototype jetable qu'on
  s'apprête à remplacer.
- On veut poser l'intention **avant** de compiler.

Si le code existe déjà et fait autorité → jouer **ADOPT** à la place.

---

## Pré-requis

- Un répertoire de travail (le futur repo `source/`, ou le repo unique en
  monorepo).
- Un interlocuteur humain disponible pour l'interview (GENESIS est dialogué).

---

## Étapes

### 1. Interview / brainstorm

Dialoguer avec l'humain pour dégager, dans l'ordre :

1. **Le pourquoi** — quel problème, pour qui, pourquoi maintenant. → `vision.md`.
2. **Les grands choix** — topologie (monorepo vs détaché, voir
   `docs/METHODOLOGY.md`), langages, contraintes dures (confidentialité,
   offline, coût…). → `architecture.md`.
3. **Le quoi/quand** — les premières capacités visées, ordonnées.
   → `roadmap.md` (avec une section « Someday » pour les velléités vagues).

Ne pas sur-spécifier : GENESIS pose un socle, pas une spec exhaustive. Les zones
d'ombre deviennent des **questions ouvertes** dans les modules.

### 2. Écrire `intent/`

Créer :

```
source/intent/vision.md
source/intent/architecture.md
source/intent/roadmap.md
source/intent/decisions/0001-<slug>.md   # une ADR par décision structurante
```

Les ADRs sont **append-only** : une décision = un fichier. On ne réécrit jamais
une ADR ; on en écrit une nouvelle qui la supersède (avec un lien
`Supersedes: 0001-...`).

### 3. Découper en capacités et écrire les premiers `modules/`

À partir de l'architecture, découper le produit en **capacités** (8–15 pour un
projet de taille moyenne). Pour chacune, créer `source/modules/<slug>.md` avec le
frontmatter :

```yaml
---
module: <slug>
status: draft            # le code n'existe pas encore
governs:                 # chemins PRÉVUS (peuvent ne pas exister encore)
  - src/.../
sync-repo: .             # "." en monorepo ; sinon chemin du repo code
last-sync:               # vide tant que rien n'est compilé
depends: [<autre-module>]
---
```

Corps de chaque module :

- **Contrat de comportement** — ce que la capacité garantit, en prose.
- **Critères d'acceptation** — liste vérifiable (chaque item testable).
- **Non-buts** — ce que la capacité ne fait explicitement PAS.
- **Questions ouvertes** — les zones à trancher pendant COMPILE.

### 4. Scaffolder le repo code + amorçage agent

- Créer la structure de répertoires prévue dans `governs`.
- Déposer à la racine du repo code `CLAUDE.md` **et** `AGENTS.md` (identiques),
  portant :
  1. le pointeur « lis `source/SOURCE.md` d'abord » ;
  2. la **directive INTAKE** complète (voir `docs/METHODOLOGY.md` / le rappel
     dans `AUDIT.md`) ;
  3. le **contrat de commit** de la topologie choisie (voir `docs/METHODOLOGY.md`).

> En topologie détachée (repo code public séparé), l'amorçage côté repo code
> reste **générique** : il décrit « lis les sources markdown si elles te sont
> fournies » **sans nommer ni pointer** le repo privé (contrainte de
> confidentialité).

### 5. Premier commit

Commiter le socle. En monorepo, `source/` et le scaffold code voyagent dans le
même commit. En détaché, commiter `source/` dans le repo privé.

---

## Sortie attendue

- `intent/` peuplé (vision, architecture, roadmap, ≥1 ADR).
- `modules/` : 8–15 specs en `status: draft`.
- Repo code scaffoldé avec `CLAUDE.md` + `AGENTS.md`.
- **Pas encore** de `SOURCE.md` fiable : il sera généré par le premier AUDIT une
  fois du code compilé (jouer COMPILE puis AUDIT).

---

## Enchaînement

`GENESIS` → (`COMPILE` sur les modules `draft` prêts) → `AUDIT` (génère
`SOURCE.md`).
