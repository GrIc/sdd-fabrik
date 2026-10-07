# SDD-Fabrik workflow

How a change is made in this repo. This file belongs to [SDD-Fabrik](https://github.com/GrIc/sdd-fabrik) and is
replaced on upgrade: project rules go in [coding-style.md](coding-style.md) or in `AGENTS.md` outside the kit block.

## Where things live

A file is created when first needed.

| Topic | Home |
|---|---|
| The project's map: system, modules | `sdd/README.md` |
| Why, for whom, success, non-goals | `sdd/product.md` |
| Look, accessibility, voice | `sdd/design.md` |
| Code, tests, docs and commit conventions, project commands | `sdd/coding-style.md` |
| System design, data, deployment | `sdd/architecture.md` |
| Decisions, with what was rejected | `sdd/decisions.md` |
| Status, phases, someday | `sdd/roadmap.md`, and nowhere else |
| What a module must do | `sdd/modules/<module>.md` |
| Bugs and ideas caught in passing | `sdd/issues/<slug>.md` |
| Acceptance, implementation facts | test names, code |

## GENESIS: a new project

Ask the human, one question at a time: the first useful slice, for whom, the non-goals, whose coding conventions to
follow, and the qualities that matter here: accessibility, devices, performance, cost, carbon. Write only what the
first slice needs: `README.md`, `coding-style.md` with its first rules and commands, one contract per module the
slice touches. Scaffold with the stack's generator, then strip what the slice does not use. The first delivery follows
COMPILE.

## ADOPT: existing code

Read the existing instructions, README, tests and entry points. Keep the behavior and the existing checks. Write the
map, a coding style drawn from what the code already does, and contracts only where the intent is not obvious from code
and tests, with `governs` paths checked by `git ls-files`. Changes you would like to make become issues, not contract
text.

## COMPILE: one batch

1. Read the intent, the contract, its tests and the code. Ask the open intent questions. When behavior depends on real
   input, measure it with a throwaway script first, and stop at the approach that covers 95 to 98% of real cases.
2. Build the smallest coherent change, with the tests it deserves ([coding style](coding-style.md#tests)). Run the
   project's checks.
3. Update the contract if the intent changed. Otherwise propose `Spec-Unchanged(<module>): <reason>`.
4. Read the whole diff against the coding style and delete what does not earn its place.
5. Stage the batch, write the proposed message with `Agent-Assisted: yes` to `.git/sdd/message`, and run
   `bash sdd/bin/sdd audit --message .git/sdd/message`.
6. Review: spawn a fresh reviewer, a new context that did not write the batch. It reads the staged diff, the contract
   and the coding style, reports findings with their evidence, and writes the quiz once the findings are fixed. If your
   tool cannot spawn one, write the quiz yourself and say so.
7. Hand off: purpose, the decisions you took alone, evidence, limits and a short "understand this" for the human. The
   human reads the diff, answers `QUIZ.md` and commits with `git commit -F .git/sdd/message`.

## Quiz

`bash sdd/bin/sdd fingerprint` prints the batch fingerprint and the two paths to write. The reviewer writes:

- `QUIZ.md` at the repo root (kept out of git): three questions that make the human predict, not recognize. What does
  this return for that input, which line enforces this rule, what breaks if this line goes, why this option over that
  one. Each question cites a changed `file:line` and has 2 to 4 boxes.

  ```markdown
  # Quiz for this batch

  Tick one box per question, save, then commit.

  ## 1. What does parse_duration return for "2 hours 30"?

  See src/durations.py:14

  - [ ] 150 minutes
  - [ ] 2 hours
  - [ ] an error
  ```

- the key, at the printed path: the fingerprint, then one line per question with the right letter (A for the first
  box) and the explanation shown after a wrong answer.

  ```
  3f2a9c...
  A "30" after hours counts as minutes.
  ```

The commit fails until all three boxes are right; each wrong answer shows its explanation. If the batch changed
since the quiz was written, the hook only warns: ask for a new quiz when the changes touch what it asks.

## Contracts

```markdown
---
governs: [src/search.py, src/templates/search/, tests/test_search.py]
depends: [tags]
---
# search

Intent, guarantees, constraints and non-goals. Links to the tests that prove them.
```

Govern the code that carries the intent: leave CI, dependency and build files out unless the module is about them.
At most 50 lines. Intent, never facts that a refactor keeping the behavior would change (SQL shapes, file layouts,
JSON formats). Acceptance lives in the test names: no checkboxes, no status. `depends` names the modules this one
builds on; check their tests pass before you do.

## Decisions

`sdd/decisions.md`, numbered. Each entry gives the choice, its evidence, and a `Rejected:` line with the alternatives
and why. Numbers never change and no entry is deleted: a superseded entry shrinks to `n. Superseded by m.`, and git
keeps its text.

## Commit trailers

| Trailer | Written by | Effect |
|---|---|---|
| `Agent-Assisted: yes` | the agent, in its proposed message | drift and size block, the quiz is required |
| `Spec-Unchanged(<module>): <reason>` | the author | spares that module from the drift check and the one-module limit |
| `Large-Batch: <reason>` | the human | lifts the size limits (`MAX_LINES` in `sdd/bin/sdd`, one module) |
| `Quiz: 3/3` | the hook | records the passed quiz |

A co-author address listed in `AGENT_EMAILS` (top of `sdd/bin/sdd`) counts as `Agent-Assisted: yes`. Any other
commit is the human's own: drift and size only warn. CI replays drift and size on every pushed commit, never the quiz.
Everything here can be bypassed: it is there to help, not to lock.

The hook compares the batch with HEAD: to amend an agent batch that changes files, run `git reset --soft HEAD^` and
commit again. Merge pull requests with a merge commit or a rebase: a squash adds the batches up, so the size check
fails on the default branch, or it drops their trailers.

## Upgrade (for agents)

1. `git ls-remote --tags --refs https://github.com/GrIc/sdd-fabrik` gives the latest `vX.Y.Z`; the deployed one is on
   the first line of `sdd/README.md`.
2. `git clone https://github.com/GrIc/sdd-fabrik <tmp>`; `git -C <tmp> diff <old> <new> -- template/` shows what
   changed. Then `git -C <tmp> checkout -q <new>`.
3. Replace `sdd/WORKFLOW.md` and `sdd/bin/sdd` with the new tag's (keep the project's changes to `MAX_LINES` and
   `AGENT_EMAILS`), and the block between `<!-- sdd-fabrik -->` and `<!-- /sdd-fabrik -->` in `AGENTS.md`. Apply the
   other template changes the diff shows, such as the CI workflow, with the human. Put the new tag on the first line
   of `sdd/README.md`. Delete `<tmp>`.
4. Tell the human what changed and propose a commit message without `Agent-Assisted` or an agent's `Co-Authored-By`:
   the files come from the kit.

## Uninstall (for agents)

Remove `sdd/bin/`, `sdd/WORKFLOW.md`, the `commit-msg` hook if it holds `# sdd-fabrik` (or the
`bash sdd/bin/sdd hook` line added to another hook), the `sdd/bin/sdd` line of `.gitattributes`, the `/QUIZ.md` line
of `.git/info/exclude`, the block between the markers in `AGENTS.md`, the `@AGENTS.md` line of `CLAUDE.md`, the
`.github/workflows/sdd.yml` step, and the deny rules added to your tool. Ask the human before removing the project's
own knowledge: `sdd/README.md`, `coding-style.md`, `modules/`, `decisions.md` and the other topic files.
