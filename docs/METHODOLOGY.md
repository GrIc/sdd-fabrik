# Méthodologie Sourcier

La version longue. Le [README](../README.md) suffit pour démarrer ; ce document
explique *pourquoi* le kit est construit ainsi et *comment* chaque pièce
s'articule.

---

## 1. Le pari

Deux pathologies rongent le couplage entre une spec et le code qu'elle est
censée décrire :

1. **La spec ment sur le code.** Une spec suppose une infrastructure jamais
   livrée, ou décrit un état déjà dépassé. L'agent compile à partir d'une carte
   fausse et doit tout réécrire au moment de l'implémentation.
2. **La capture au fil de l'eau se perd.** Un bug, une contrainte, une décision
   mentionnés en passant n'atterrissent dans aucun fichier. L'intention
   émergente s'évapore entre deux sessions. C'est le **trou commun à tous les
   kits spec-driven** : ils formalisent une spec posée, pas l'intention
   émergente.

Sourcier est un kit **purement markdown, sans dépendance outil** qui matérialise
quatre propriétés qu'aucun kit existant ne combine :

- **sources vivantes** — la spec est la source de vérité, révisée en continu ;
- **capture INTAKE** — l'intention émergente est happée au fil de la
  conversation ;
- **multi-agent agnostique** — Claude Code, Antigravity, ou tout agent lisant du
  markdown et lançant `git` ;
- **bootstrap standardisé** — quatre protocoles reproductibles.

Le seul point d'injection propre à chaque outil est le fichier d'amorçage racine
(`CLAUDE.md` pour Claude Code, `AGENTS.md` pour les autres). Le reste — topologie,
protocoles, frontmatter — est lisible et exécutable par tout agent.

**Humains et agents partagent la source.** `source/` n'est pas un artefact de
machine : un contributeur humain l'édite directement (une ADR, un critère
d'acceptation), un agent lit et écrit le même arbre. La frontière n'est pas
humain/agent mais **édité/généré** (voir §2).

---

## 2. Structure de l'arbre `source/`

```
source/
  SOURCE.md              # index GÉNÉRÉ par AUDIT — jamais édité à la main.
                         #   Table des modules, statuts, santé de sync.
  intent/                # NIVEAU 1 — stable, lent à changer :
    vision.md            #   le pourquoi.
    architecture.md      #   les grands choix structurants.
    roadmap.md           #   le quoi/quand, avec une section « Someday ».
    decisions/           #   ADRs légers, append-only (une décision = un fichier).
  modules/               # NIVEAU 2 — vivant : 1 spec par capacité,
                         #   frontmatter `governs` (voir §3).
  issues/
    features/            # user stories / epics.
    bugs/                # bugs et limitations connues.
  protocols/             # GENESIS.md, ADOPT.md, COMPILE.md, AUDIT.md.
  reports/               # rapports d'audit datés, générés.
```

**Deux niveaux, deux tempos.** `intent/` est le socle stable — on y touche
rarement, et les décisions y sont *append-only* (on n'efface pas une ADR, on en
écrit une nouvelle qui la supersède). `modules/` est le tissu vivant — une spec
par capacité, révisée à chaque cycle de compilation.

**Généré vs édité.** `SOURCE.md` et `reports/` sont **générés** par AUDIT.
Tout le reste est **édité** (par un humain ou un agent suivant un protocole).
Cette frontière est ce qui empêche « la carte qui ment » : la vérité de l'état
(quels modules, quel statut, quelle santé de sync) est **dérivée** du frontmatter
et de l'état git, jamais re-saisie à la main.

### Pourquoi pas de manifeste central édité à la main

Un fichier pivot que chaque contributeur met à jour manuellement devient (a) un
**goulot** (tout passe par lui), (b) une **source de conflits** de merge, et (c)
surtout **une carte qui ment** dès que quelqu'un oublie de le synchroniser.
`SOURCE.md` est donc **généré**, jamais édité.

### Couplage au niveau capacité, pas au niveau symbole

Sourcier couple une spec à une **capacité** (une responsabilité cohérente),
`governs` pointant un ou plusieurs répertoires. Attacher une spec à chaque
fonction/module physique (mapping 1:1) coûte plus d'entretien que la valeur
rendue. 8–15 modules suffisent pour un projet de taille moyenne.

---

## 3. Frontmatter d'une spec module et détection de dérive

### 3.1 Frontmatter

```yaml
---
module: <slug>          # identifiant stable de la capacité
status: draft | compiled | drifted | deprecated
governs:                # chemins du repo code que cette spec régit (base du diff)
  - src/.../
  - tests/.../
sync-repo: .            # "." = monorepo ; sinon chemin vers le repo code
last-sync: abc1234      # SHA du repo code au dernier point de sync
depends: [<autre-module>]   # modules amont (vérifiés avant COMPILE)
---
```

`governs` accepte les deux écritures YAML : liste en bloc (ci-dessus) ou inline
`governs: [src/a/, tests/a/]`. Le script d'audit gère les deux.

Le corps de la spec contient : le **contrat de comportement**, les **critères
d'acceptation**, les **non-buts**, et les **questions ouvertes**.

### 3.2 Cycle de vie du statut

- `draft` — spec écrite, code pas encore compilé (sortie typique de GENESIS).
- `compiled` — code livré et conforme, `last-sync` à jour (sortie de COMPILE ;
  aussi l'état d'entrée produit par ADOPT sur du brownfield).
- `drifted` — le code a bougé sous la spec sans que la spec suive (détecté par
  AUDIT).
- `deprecated` — capacité retirée ; conservée pour l'historique.

### 3.3 Détection de dérive — MÉCANIQUE, pas relecture

Le cœur du kit. La dérive n'est **jamais** jugée par relecture ; elle est
**calculée** :

```bash
git -C <sync-repo> diff --stat <last-sync>..HEAD -- <governs...>
```

**Règle.** Diff non vide **et** spec inchangée depuis `last-sync` ⇒ le module
passe `status: drifted`. Le champ `last-sync` est **remis à jour par COMPILE en
fin de travail** (nouveau HEAD du repo code une fois le code conforme). Ce
mécanisme est déterministe : deux agents sur le même état produisent le même
verdict. Un frontmatter structuré rend le diff calculable, là où des protocoles
en prose seuls rendraient l'audit non déterministe (deux agents, deux verdicts).

Le script complet, gérant les deux écritures de `governs`, est dans
[`../protocols/AUDIT.md`](../protocols/AUDIT.md) §1.

---

## 4. Réflexe INTAKE — la capture au fil de l'eau

C'est la réponse au trou commun des kits spec-driven. L'INTAKE est une
**directive permanente** vivant dans `CLAUDE.md` / `AGENTS.md` — le **seul point
d'injection chargé à chaque session, pour tous les agents**. Elle transforme
chaque conversation en opportunité de capture, sans jamais interrompre la tâche
en cours.

### La directive

Quand l'utilisateur décrit un bug, une feature, une décision ou une contrainte —
**même en passant** :

1. **Chercher** dans `source/issues/` via l'index `SOURCE.md`.
2. **Si ça existe** → **enrichir** le fichier existant.
3. **Sinon** → **créer un stub** `status: captured` : frontmatter minimal, une
   phrase de description, un lien vers la conversation.
4. **Le dire en une ligne**, puis **REPRENDRE** immédiatement la tâche en cours.

### Poids et rythme

- Un stub est **léger (~30 s)**, ce n'est **pas** une spec complète.
- `status: captured` = **file d'attente**. Le stub sera raffiné plus tard (par
  GENESIS/COMPILE ou une passe de tri dédiée).
- « Il faudra un jour X » (velléité vague, pas une demande) → **une ligne dans
  `roadmap.md` section « Someday »**, pas un fichier.

### Filets de sécurité

- **AUDIT signale** les stubs `captured` qui **vieillissent** (file qui stagne).
- **Un fix commité sans issue correspondante** est de la dérive, détectée par le
  contrat de commit.

### Exemple de stub

```markdown
---
status: captured
kind: bug
captured-on: 2026-07-05
source: conversation 2026-07-05
---
Le spinner tourne sans consommer de tokens quand le stream se coupe tôt.
À raffiner : reproduire, isoler la condition de coupure.
```

---

## 5. Les quatre protocoles

Fichiers markdown que **tout agent exécute**. Sans dépendance outil : une
séquence d'actions markdown + git. La source canonique est
[`../protocols/`](../protocols/) ; `template/source/protocols/` en est une copie.

- **GENESIS** — greenfield : interview → `intent/` → premiers `modules/` en
  `draft` → scaffold du code + amorçage agent.
- **ADOPT** — brownfield : lit le code, en déduit les capacités, écrit les
  `modules/` en `compiled` avec `last-sync = HEAD` (le code est la vérité de
  départ), migre les docs vers `intent/`.
- **COMPILE** — production de code : vérifie `depends`, planifie, délègue en TDD,
  vérifie les critères d'acceptation, met à jour la spec + `last-sync` (même
  commit en monorepo).
- **AUDIT** — le gardien : diff mécanique par module → passe les dérivés en
  `drifted`, régénère `SOURCE.md`, écrit un rapport daté, signale les captures
  non triées.

### Règle de réconciliation

Quand code et spec divergent, AUDIT (ou le gardien humain) tranche selon :

> **LE CODE GAGNE SUR LES FAITS, L'INTENT GAGNE SUR LE BUT.**

- Le code a divergé pour de **bonnes raisons** (le monde réel a corrigé un *fait*
  que la spec décrivait mal) → **backfiller la spec** pour qu'elle dise la vérité
  du code, puis remettre `last-sync = HEAD` et repasser en `compiled`.
- Le code a **trahi l'intention** (il fait autre chose que ce que la vision
  voulait) → **rouvrir une issue** ; c'est le code qui doit revenir dans le rang,
  pas l'intention qu'on réécrit.

Le discernement porte sur la **nature** de la divergence : désaccord de *fait*
(le code sait mieux) vs désaccord de *but* (l'intention sait mieux).

---

## 6. Contrat de commit et topologies

### 6.1 Monorepo (défaut)

`source/` et le code dans le même repo (`sync-repo: .`). Un commit qui touche un
chemin listé dans un `governs` **doit** toucher la spec correspondante, **dans le
même commit**. Sinon, il porte le trailer :

```
No-Spec-Impact: <raison>
```

pour les commits légitimes sans impact spec (refactor mécanique, typo, bump de
dépendance). L'enforcement dur (hook `pre-commit` / CI) est **possible mais
optionnel** : à défaut, **AUDIT rattrape**. C'est de la **défense en
profondeur** — contrat de commit en première ligne, AUDIT en filet. Une CI
bloquante obligatoire transformerait la discipline en friction et inciterait au
contournement (`--no-verify`, trailers bidon) ; elle n'est utilisée que là où
l'équipe la veut vraiment.

### 6.2 Détachée

Deux repos (ex. source privée / code public). Ils ne partagent pas d'historique :
le contrat même-commit **ne peut pas** être imposé mécaniquement, il est donc
**affaibli**. En conséquence :

- **AUDIT devient la défense principale.**
- Le **rythme d'audit est plus soutenu** (idéalement à chaque session de
  compilation côté code, au minimum hebdomadaire), pour rattraper vite la dérive
  que le commit ne peut pas prévenir.
- Chaque module porte `sync-repo: <chemin-du-clone-code>` (relatif ou absolu).

**Confidentialité (si la source est privée et le code public).** Le repo public
ne référence **jamais** le repo source privé. La directive INTAKE vit dans le
`CLAUDE.md` du repo **source** (privé). Côté code public, la formulation
d'amorçage reste **générique** : « lis les sources markdown si elles te sont
fournies », **sans nommer** ni pointer le repo privé.

---

## 7. Règles de source (anti-mensonge)

- **Ne recopie jamais un fait dérivable du code** dans la source : ni compteur de
  tests, ni LOC, ni effectifs. Ces chiffres pourrissent. L'intent porte des
  pointeurs ; les faits sont **générés** (`SOURCE.md`, rapports d'audit).
- Une spec qui marque une dépendance livrée (`✅`) doit pointer vers du code
  **mergé**, pas une proposition sœur non intégrée.
