# <projet> — source vivante (méthode Sourcier)

> Le code de ce projet est un **artefact compilé** d'une source markdown
> vivante. Cette source (`source/`) porte l'intention, les contrats de
> comportement (modules), les issues, les décisions. Le code en découle.
>
> Ce fichier est le point d'injection chargé à **chaque session** : il porte le
> pointeur vers la source, le réflexe INTAKE, et le contrat de commit.
> Humains et agents lisent la **même** source de vérité.

## Lis d'abord

**`source/SOURCE.md`** — l'index généré (table des modules, statuts, santé de
sync, file des issues). Point d'entrée unique. Ne l'édite jamais à la main : il
est **régénéré** par le protocole AUDIT.

Puis, selon ton besoin :
- `source/intent/` — vision, architecture, roadmap, `decisions/` (ADRs).
- `source/modules/` — un contrat par capacité, frontmatter `governs` mappé au
  code.
- `source/protocols/` — GENESIS, ADOPT, COMPILE, AUDIT. Les « programmes » que
  tu exécutes.

## Réflexe INTAKE — permanent, non négociable

Quand l'utilisateur décrit **un bug, une idée de feature, une décision ou une
contrainte** — même en passant, même si la tâche du moment est autre :

1. Cherche dans `source/issues/` (via l'index `SOURCE.md`) si ça existe déjà.
2. **Existe** → enrichis le fichier existant (nouveau symptôme, contexte).
3. **N'existe pas** → crée un stub `status: captured` : frontmatter minimal, une
   phrase de contexte, un lien vers la conversation. ~30 secondes, pas une spec
   complète.
4. Dis-le en **une ligne** (« bug capturé → `issues/bugs/x.md` ») et **reprends
   la tâche en cours**.

« Il faudra un jour X » → une ligne dans `intent/roadmap.md` (section *Someday*),
pas un fichier. Les stubs `captured` sont une file d'attente ; GENESIS/COMPILE
les raffinent plus tard ; AUDIT signale ceux qui vieillissent.

## Contrat de commit — MONOREPO (défaut)

`source/` et le code vivent dans le **même repo**. Chaque module porte
`sync-repo: .`. Donc :

- Un commit qui touche un chemin listé dans un `governs` **doit** toucher la spec
  correspondante, **dans le même commit**. Sinon, il porte le trailer
  `No-Spec-Impact: <raison>` (refactor mécanique, typo, bump de dépendance).
- Enforcement possible en dur (hook `pre-commit` / CI) mais **optionnel** : à
  défaut, **AUDIT rattrape** (défense en profondeur).
- Règle de réconciliation : **le code gagne sur les faits, l'intent gagne sur le
  but.** Backfille la spec quand le code a divergé pour de bonnes raisons ;
  rouvre une issue quand le code a trahi l'intention.

<!--
  VARIANTE DÉTACHÉE (deux repos : source privée / code public séparé).
  À activer À LA PLACE du bloc monorepo ci-dessus si le code vit dans un autre
  repo (ex. confidentialité : source privée, code public).

  Dans chaque module : `sync-repo: ../code-repo` (chemin du clone local du code).

  ## Contrat de commit — topologie détachée

  Le repo code est **séparé** : le contrat « même commit » (spec+code atomiques)
  est impossible. Donc :

  - **AUDIT est la défense principale** contre la dérive. À jouer à rythme
    soutenu (idéalement à chaque session de compilation côté code, au minimum
    hebdomadaire). Il fait le diff mécanique
    `git -C ../code-repo diff --stat <last-sync>..HEAD -- <governs>` par module
    et remet les statuts à jour.
  - Un fix côté code **sans** issue/module correspondant ici = dérive à
    rattraper.
  - Même règle de réconciliation (code gagne sur les faits, intent sur le but).

  ## Confidentialité (si la source est privée et le code public)

  Le repo public code ne référence **jamais** ce repo. Toute amorce côté public
  reste générique (« lis les sources markdown si fournies »), sans nommer ni
  pointer le repo source privé ni `source/`.
-->
