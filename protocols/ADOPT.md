# Protocole ADOPT — adopter un projet existant (brownfield)

> Rétro-ingénierie d'un code **déjà là**. ADOPT lit le code, en déduit les
> capacités, et rédige les specs `modules/` en `status: compiled` avec
> `last-sync = HEAD` — car **le code est la vérité de départ**.
>
> Exécutable par tout agent sachant lire du markdown et lancer `git`.

---

## Quand jouer ADOPT

- Le code existe et fait autorité, mais il n'y a pas (ou plus) de specs vivantes
  couplées.
- On veut backfiller `source/modules/` contre un code déjà en production.

Si le code n'existe pas encore → jouer **GENESIS** à la place.

---

## Pré-requis

- Un accès au repo code. En **détaché**, un clone local du repo code en
  **lecture seule** (ADOPT ne modifie JAMAIS le repo code). En **monorepo**, le
  code vit dans le même repo (`sync-repo: .`).
- Le SHA du `HEAD` du repo code au moment du backfill (il servira de
  `last-sync`).

```bash
# <code-repo> = "." en monorepo ; sinon le chemin du clone (ex. ../code-repo)
git -C <code-repo> rev-parse --short HEAD     # -> last-sync
git -C <code-repo> rev-parse HEAD             # sha long pour traçabilité
```

---

## Étapes

### 1. Lire le code et en déduire les capacités

- Lire le point d'entrée d'onboarding s'il existe (README, ONBOARDING,
  architecture) — il donne souvent la carte des modules.
- Parcourir l'arbre du code : repérer les frontières naturelles (un service, un
  moteur, une couche transport, une couche UI…).
- Regrouper en **capacités** (pas en fichiers physiques) : 8–15 modules pour un
  projet de taille moyenne. Une capacité = une responsabilité cohérente, qui
  `governs` un ou plusieurs répertoires **réels et vérifiés**.

> Vérifier chaque chemin `governs` contre l'arbre réel :
> ```bash
> git -C <code-repo> ls-files -- <chemin> | head    # non vide = chemin valide
> ```

### 2. Rédiger les specs `modules/`

Pour chaque capacité, créer `source/modules/<slug>.md` :

```yaml
---
module: <slug>
status: compiled          # le code est la vérité de départ
governs:                  # chemins RÉELS, vérifiés à l'étape 1
  - src/.../
  - tests/.../
sync-repo: .              # "." en monorepo ; sinon chemin du clone local
last-sync: <sha-court>    # HEAD du repo code au moment du backfill
depends: [<autre-module>]
---
```

Corps (déduit du code, pas inventé) :

- **Contrat de comportement** — ce que la capacité fait *réellement*
  aujourd'hui, tel que le code l'atteste.
- **Critères d'acceptation** — invariants vérifiables que le code respecte
  (rejouables comme tests de non-régression).
- **Non-buts** — ce que le code ne fait délibérément pas (utile pour éviter que
  COMPILE ne réintroduise du scope écarté).
- **Questions ouvertes** — dettes, zones floues, TODO repérés en lisant.

> Règle de fidélité : ADOPT **décrit le code tel qu'il est**, pas tel qu'on
> voudrait qu'il soit. Toute divergence entre le code et l'intention se règle
> plus tard via la règle de réconciliation d'AUDIT (« le code gagne sur les
> faits, l'intent gagne sur le but »). Ne pas « corriger » la spec pour qu'elle
> décrive un futur souhaité — cela recrée « la carte qui ment ».

### 3. Migrer les docs existants vers `intent/`

- Vision / raison d'être → `intent/vision.md`.
- Choix structurants / architecture → `intent/architecture.md`.
- Roadmap / journal → `intent/roadmap.md`, `intent/journey.md`.
- Décisions historiques → `intent/decisions/` (une ADR par décision).
- Assessments / recherches datés → `reports/`.

Utiliser `git mv` pour **préserver l'historique** quand les fichiers existent
déjà dans le repo source.

### 4. Pas de `SOURCE.md` à la main

Ne **pas** écrire `SOURCE.md` manuellement. Une fois les modules backfillés,
jouer **AUDIT** : il génère `SOURCE.md` et produit le premier rapport daté. Cette
première passe AUDIT sert aussi de **contrôle** du backfill (governs/last-sync
mal renseignés apparaissent immédiatement).

---

## Sortie attendue

- `modules/` : 8–15 specs en `status: compiled`, chacune avec un `governs`
  vérifié et un `last-sync` valide.
- `intent/` peuplé à partir des docs migrés.
- Le repo code **inchangé** (ADOPT est lecture seule dessus).

---

## Enchaînement

`ADOPT` → `AUDIT` (génère `SOURCE.md` + premier rapport, contrôle le backfill)
→ ensuite `COMPILE` au fil des issues.
