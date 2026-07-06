# Sourcier — un kit de sources markdown vivantes → code compilé par agents

**Le code est un artefact compilé d'une source markdown vivante.** Tu écris et
révises l'intention (vision, contrats de comportement, décisions, issues) en
markdown ; des agents (Claude Code, Antigravity, ou autres) *compilent* ce code
à partir de cette source. La source reste la vérité de référence, révisée en
continu — pas une spec figée générée une fois.

Zéro dépendance outil : que du markdown et `git`. **Humains et agents lisent et
écrivent la même source** — un contributeur humain ouvre `source/`, un agent lit
le même arbre via `CLAUDE.md` / `AGENTS.md`. Rien n'est réservé à l'un ou à
l'autre.

Ce que Sourcier ajoute par rapport aux kits spec-driven classiques :

- **Sources vivantes** — la spec est révisée en continu, elle ne se périme pas.
- **Détection de dérive mécanique** — `git diff`, pas relecture humaine :
  quand le code bouge sous une spec, c'est **calculé**, pas jugé.
- **Réflexe INTAKE** — l'intention émergente (un bug, une idée lâchés en
  passant) est happée au fil de la conversation, pas seulement les specs
  délibérées.
- **Quatre protocoles** reproductibles, exécutables par n'importe quel agent.

---

## Démarrer

Copie le squelette [`template/`](template/) à la racine de ton projet, puis
donne à ton agent le prompt d'amorçage correspondant à ton cas.

### Mode A — nouveau projet (greenfield) → protocole GENESIS

Tu pars d'une idée, aucun code encore.

```
cp -r <chemin-vers-sdd-fabrik>/template/. .
```

Puis, à ton agent (Claude Code, Antigravity, ou autre) :

> Lis `source/protocols/GENESIS.md` et joue-le avec moi. Interviewe-moi pour
> dégager la vision, l'architecture et la roadmap ; écris `source/intent/` et
> les premiers `source/modules/` en `status: draft` ; scaffolde le code et son
> amorçage agent. Ne sur-spécifie pas.

### Mode B — projet existant (brownfield) → protocole ADOPT

Tu as déjà du code qui fait autorité.

```
cp -r <chemin-vers-sdd-fabrik>/template/. .     # sans écraser ton code
```

Puis, à ton agent :

> Lis `source/protocols/ADOPT.md` et joue-le sur ce projet. Lis le code, déduis
> les capacités, écris les specs `source/modules/` en `status: compiled` avec
> `last-sync = HEAD`, migre les docs existants vers `source/intent/`. Le code
> est la vérité de départ : décris-le tel qu'il est. Puis joue AUDIT pour
> générer `source/SOURCE.md`.

Une fois amorcé, le cycle de travail est : **COMPILE** (produire du code depuis
une spec/issue prête) et **AUDIT** (le gardien : détecte la dérive, régénère
l'index). Les quatre protocoles sont dans
[`template/source/protocols/`](template/source/protocols/).

---

## Topologies

- **Monorepo (défaut)** — `source/` et le code dans le même repo. La spec et le
  code voyagent dans le **même commit**. `sync-repo: .`.
- **Détachée** — deux repos (ex. source privée / code public). Le même-commit
  est impossible, donc **AUDIT** devient la défense principale. Chaque module
  porte `sync-repo: <chemin-du-clone-code>`.

Le [`template/CLAUDE.md`](template/CLAUDE.md) est en mode monorepo par défaut ;
la variante détachée y est documentée en commentaire, prête à activer.

---

## Où vit quoi

- [`template/`](template/) — le squelette à copier dans ton projet.
- [`protocols/`](protocols/) — **la source canonique** des quatre protocoles
  (GENESIS, ADOPT, COMPILE, AUDIT). `template/source/protocols/` en est une
  **copie**. Si tu modifies un protocole, édite `protocols/` puis recopie ; ne
  maintiens pas les deux à la main.
- [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) — la version longue : structure de
  `source/`, frontmatter, anti-drift mécanique, INTAKE, réconciliation,
  topologies.
- [`docs/QUICKSTART.md`](docs/QUICKSTART.md) — le pas-à-pas des deux modes.

## Licence

MIT — voir [`LICENSE`](LICENSE).
