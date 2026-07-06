# Quickstart — les deux modes de démarrage

Sourcier a deux points d'entrée selon que ton projet a déjà du code ou non.
Dans les deux cas : copie le squelette, puis lance le protocole avec ton agent.

Prérequis : `git`, et un agent capable de lire du markdown et lancer `git`
(Claude Code, Antigravity, ou autre).

---

## Étape commune — copier le squelette

Depuis la racine de ton projet (nouveau ou existant) :

```bash
cp -r <chemin-vers-sdd-fabrik>/template/. .
git add source CLAUDE.md AGENTS.md
```

> Le squelette apporte `source/` (avec les 4 protocoles et un `SOURCE.md`
> placeholder) et les amorces `CLAUDE.md` / `AGENTS.md` (monorepo par défaut).
> Sur un projet existant, `cp -r template/. .` **n'écrase pas** ton code — il
> ajoute `source/` à côté. Si tu as déjà un `CLAUDE.md`, fusionne à la main
> plutôt que d'écraser.

Choisis ta topologie :

- **Monorepo (défaut)** : rien à changer, les modules utiliseront `sync-repo: .`.
- **Détachée** (source privée / code dans un autre repo) : dans
  `CLAUDE.md`/`AGENTS.md`, active le bloc « variante détachée » commenté, et
  utilise `sync-repo: <chemin-du-clone-code>` dans chaque module.

---

## Mode A — nouveau projet (GENESIS)

Tu pars d'une idée, aucun code encore.

1. Donne à ton agent :

   > Lis `source/protocols/GENESIS.md` et joue-le avec moi. Interviewe-moi pour
   > dégager vision / architecture / roadmap ; écris `source/intent/` et les
   > premiers `source/modules/` en `status: draft` ; scaffolde le code et son
   > amorçage agent. Ne sur-spécifie pas.

2. L'agent t'interviewe, écrit `intent/`, découpe en 8–15 capacités
   (`modules/` en `draft`), et scaffolde la structure du code.

3. Cycle de travail :
   - **COMPILE** un module `draft` prêt : `Lis source/protocols/COMPILE.md et
     compile le module <slug>.`
   - **AUDIT** pour générer l'index : `Lis source/protocols/AUDIT.md et joue-le.`
     → produit `source/SOURCE.md` et un rapport daté dans `source/reports/`.

---

## Mode B — projet existant (ADOPT)

Tu as déjà du code qui fait autorité.

1. Donne à ton agent :

   > Lis `source/protocols/ADOPT.md` et joue-le sur ce projet. Lis le code,
   > déduis les capacités, écris les specs `source/modules/` en
   > `status: compiled` avec `last-sync = HEAD`, migre les docs existants vers
   > `source/intent/`. Le code est la vérité de départ : décris-le tel qu'il
   > est, ne le corrige pas dans la spec.

2. L'agent lit le code, écrit les `modules/` en `compiled` (chaque `governs`
   vérifié contre l'arbre réel, `last-sync` = HEAD), et migre les docs.

3. Joue **AUDIT** immédiatement : il génère `source/SOURCE.md` et un premier
   rapport, ce qui **contrôle** le backfill (un `governs` ou `last-sync` mal
   renseigné apparaît tout de suite).

   > Lis `source/protocols/AUDIT.md` et joue-le : génère `source/SOURCE.md` et le
   > premier rapport.

---

## Ensuite — le rythme de croisière

- **INTAKE** tourne en permanence : dès que tu mentionnes un bug/feature/décision
  en passant, l'agent crée un stub `captured` dans `source/issues/` et reprend.
- **COMPILE** produit le code depuis une spec/issue prête et remet `last-sync`
  à jour.
- **AUDIT** (hebdo / pré-release / à la demande) détecte la dérive mécaniquement,
  régénère `SOURCE.md`, écrit un rapport, signale les captures qui vieillissent.

Détails et *pourquoi* dans [`METHODOLOGY.md`](METHODOLOGY.md).
