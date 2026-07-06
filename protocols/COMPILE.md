# Protocole COMPILE — une spec / issue est prête, produire le code

> Le protocole de **production de code**. On part d'une spec module ou d'une
> issue prête, on vérifie les dépendances, on planifie, on délègue en TDD, on
> vérifie les critères d'acceptation, et on **remet `last-sync` à jour**.
>
> Exécutable par tout agent sachant lire du markdown, lancer `git`, et écrire du
> code.

---

## Quand jouer COMPILE

- Une spec `modules/<slug>.md` en `status: draft` est prête à être compilée.
- Une issue `issues/features/<slug>.md` ou `issues/bugs/<slug>.md` est prête à
  être livrée.
- Un module `drifted` doit être ramené dans le rang (le code a trahi l'intent).

---

## Étapes

### 1. Vérifier les dépendances (`depends`)

Lire le frontmatter `depends:` de la spec. Pour chaque module amont :

- Vérifier qu'il est dans un état **exploitable** (`status: compiled`, pas
  `draft` ni `drifted`).
- Si un amont est `draft`/`drifted` et que la capacité en dépend réellement →
  **s'arrêter** : compiler d'abord l'amont, ou réduire le scope pour ne pas en
  dépendre.

### 2. Établir un plan

- Relire le **contrat de comportement**, les **critères d'acceptation**, les
  **non-buts** de la spec.
- Vérifier que la spec dit la **vérité du code actuel** (dépendances marquées
  livrées doivent pointer du code **mergé**, pas une proposition sœur). En cas
  de doute, lancer le diff mécanique (voir `AUDIT.md` §détection) sur les
  `governs` amont.
- Découper en tâches TDD (chaque tâche : un test qui échoue → code → test vert).

### 3. Déléguer en TDD

- Déléguer les tâches indépendantes à des **sous-agents** (un lot cohérent par
  sous-agent, brief précis : chemins, contrat, critères).
- Discipline **test-first** : écrire le test qui échoue avant le code.
- Respecter les **non-buts** : ne pas réintroduire du scope explicitement écarté.

### 4. Vérifier les critères d'acceptation

- Rejouer **chaque** critère d'acceptation de la spec comme une vérification
  concrète (test, exécution, observation). Un critère non vérifié = travail non
  terminé.
- Lancer la suite de tests et les gardes du repo code (linters, invariants de
  couches…).

### 5. Mettre à jour la spec + `last-sync`

Une fois le code conforme :

- Mettre la spec à jour si le comportement a évolué (contrat, critères,
  non-buts).
- Passer `status:` à `compiled`.
- **Remettre `last-sync` au nouveau `HEAD` du repo code** :
  ```bash
  # <code-repo> = "." en monorepo ; sinon le chemin du clone (ex. ../code-repo)
  git -C <code-repo> rev-parse --short HEAD   # -> nouvelle valeur de last-sync
  ```
  C'est **cette mise à jour** qui "acquitte" la dérive : après elle, le diff
  `last-sync..HEAD` sur les `governs` est vide, donc le module n'est plus
  `drifted`.
- Fermer l'issue correspondante (supprimer le fichier + retirer sa ligne d'index
  le cas échéant ; l'historique git est le journal des issues fermées).

### 6. Commiter selon le contrat de commit

**Monorepo (défaut).** La spec (`modules/`), le code, et la mise à jour
`last-sync` voyagent dans le **même commit**. Un commit touchant un chemin
`governs` doit toucher la spec ; sinon il porte le trailer
`No-Spec-Impact: <raison>`.

**Détaché.** Les deux repos ne partagent pas d'historique : le même-commit est
**impossible**. Committer alors **au plus près** :

- côté repo code : le commit de code, avec sa propre discipline ;
- côté repo source : le commit qui met à jour la spec + `last-sync`, référençant
  le SHA du repo code livré.

Comme le couplage même-commit ne peut pas être imposé en détaché, **AUDIT
devient la défense principale** et se joue à un **rythme soutenu** (idéalement à
chaque session de compilation, au minimum hebdomadaire) pour rattraper vite la
dérive.

---

## Sortie attendue

- Code livré et conforme aux critères d'acceptation.
- Spec à jour, `status: compiled`, `last-sync` = HEAD du repo code livré.
- Issue fermée.
- Commit(s) respectant le contrat de commit de la topologie.

---

## Enchaînement

`COMPILE` → `AUDIT` (vérifie qu'aucune dérive résiduelle ne subsiste, régénère
`SOURCE.md`).
