# Decisions

Numbers never change and no entry is deleted: a superseded entry shrinks to a line, and git keeps its text. Field
notes from the pilot (September 2026) are the evidence behind most entries.

## 1. A harness, not a kit of guides (2026-10-03)

Short guides plus sensors an agent meets without remembering them: a `commit-msg` hook and a CI step. The v1 kit was
1,700 lines of markdown and four protocols an agent had to remember to run; the pilot ran the method in about 270.
_Rejected:_ the four protocol files, the generated `SOURCE.md`, `last-sync` and `sync-repo`, dated audit reports.

## 2. One repo for spec and code (2026-10-03)

Drift is computed from the history that spec and code share. _Rejected:_ the detached topology (no shared history,
so drift becomes a ritual instead of a check).

## 3. Bash and git only (2026-10-03)

git is already there; on Windows, Git for Windows brings bash, sed and awk. One script of about 240 lines lands in each
project. _Rejected:_ Python (a 458-line runner was written: too heavy to copy into every project, and a runtime to
install); the pre-commit framework (needs Python and pip).

## 4. A native commit-msg hook (2026-10-03)

`install-hook` writes `.git/hooks/commit-msg` and never replaces a hook it did not write: it prints the line to add
instead. _Rejected:_ `core.hooksPath` (overrides the user's global hooks, such as a DCO check); pre-commit (decision 3).

## 5. The user's agent deploys it (2026-10-03)

The README tells an agent how to fetch a tag with git, copy without overwriting, merge an existing `AGENTS.md`, and
install the hook. _Rejected:_ an install script piped from curl (cannot merge what already exists); a manifest of raw
URLs to fetch one by one.

## 6. Kit files and project files apart (2026-10-03)

The kit owns `sdd/bin/sdd`, `sdd/WORKFLOW.md` and a marked block of `AGENTS.md`; upgrade replaces them. Everything else
is the project's. _Rejected:_ one `sdd/README.md` mixing the workflow and the project map (every upgrade conflicts).

## 7. Two weights: agents are blocked, humans warned (2026-10-03)

Constraints scale with how much of the change the human did not write. `Agent-Assisted: yes`, or a co-author address
listed in `AGENT_EMAILS` (Claude Code's, Codex's and Aider's), marks a batch. _Rejected:_ the same rules for every
commit (a human fixing a typo should not take a quiz); recognizing agents by name in `Co-Authored-By` (Claude is also
a first name, and "agent" shows up in human names and addresses); `Assisted-by:` as a mark (it can credit a human).

## 8. Drift per commit, waived per module (2026-10-03)

Each commit is checked against its parent; `Spec-Unchanged(<module>): <reason>` spares one module and says why in the
history. _Rejected:_ comparing the dates of the last commits on code and contract (a later contract touch hid an
earlier drift); a global `No-Spec-Impact` (7 in 19 pilot commits, exempting whole commits on free text).

## 9. Small batches, by count (2026-10-03)

An agent batch over 200 changed lines of code and tests, or touching more than one module, is refused unless the human
adds `Large-Batch: <reason>`. A module spared by `Spec-Unchanged` does not count: a file shared by two modules can
change for one of them. On the pilot, 10 of 12 code commits stayed under 160 lines; the two above were the hardest to
review. _Rejected:_ a guideline only; letting the agent judge its own batch (its `Spec-Unchanged` claims stand in the
message the human reads and commits).

## 10. A quiz in a file (2026-10-03)

The reviewer writes `QUIZ.md` (boxes to tick) and a key hidden in `.git/`; the hook grades it. The human commits from a
terminal or an IDE button. _Rejected:_ questions asked on the terminal (no terminal behind IDE commit buttons, nor
reliably on Windows); hashed answers (the hidden key is enough for an incentive); free-text answers (fragile matching);
review attestations checked by CI (they forbid amend, rebase and squash-merge). A batch changed since its quiz
only warns (2026-10-07): refusing it cost an LLM call for each small edit.

## 11. No status, no checkboxes in contracts (2026-10-03)

Acceptance lives in test names, and a module is done when its tests pass. _Rejected:_ `status: compiled` (forgotten
for a day on two pilot modules); acceptance boxes that restate the specs. Acceptance now lives in contracts,
linked to the tests that prove it (2026-10-07, decision 20).

## 12. Flat, optional topic files (2026-10-03)

`product.md`, `design.md`, `architecture.md`, `decisions.md` and `roadmap.md` have fixed names and are created when
first needed. _Rejected:_ an `architecture/` folder; shipping empty files that read as true.

## 13. `CLAUDE.md` imports `AGENTS.md` (2026-10-03)

One line, `@AGENTS.md`. Claude Code reads `AGENTS.md` by itself only when there is no `CLAUDE.md`. _Rejected:_ no
`CLAUDE.md` (the first one a project adds would silently hide `AGENTS.md`); a symlink (checked out as a text file on
Windows by default); a sentence asking to read `AGENTS.md` (one more read to forget).

## 14. Deny lists set by the deploying agent (2026-10-03)

The agent adds `git commit`, `git push` and `--no-verify` to its own tool's deny list and says which agents only get
the instruction. _Rejected:_ shipping a `.claude/settings.json` (binds one tool only).

## 15. One builder, one fresh reviewer when possible (2026-10-03)

A new context reads the batch and writes the quiz; without sub-agents, the builder writes it and says so. _Rejected:_
a mandatory fresh reviewer (blocks agents without sub-agents); a cast of personas in one context (role-play).

## 16. No phase-id check (2026-10-03)

"The order of the work lives only in the roadmap" stays a rule in `AGENTS.md`. _Rejected:_ grepping phase ids outside
the roadmap (the pattern differs per project and would need a setting).

## 17. A lazy agent writing human-shaped code (2026-10-03)

The agent saves effort and tokens, never thought, and tells the human what it skipped. Lists follow their natural
series (the pilot's word lists had gaps and strays found one by one in the data). Coverage follows Pareto too.
_Rejected:_ fitting lists to the data alone; a coverage target.

## 18. The kit runs its own harness (2026-10-04)

`sdd/bin/sdd` runs the template's script, and one contract, `checks`, governs the script, its CI step and its tests.
Every batch on the kit goes through drift, size and the quiz, and the owner reviews it before each commit. GitHub
reads workflows only from `.github/workflows`, so the kit's audit job repeats the template's. _Rejected:_ a copy of
the script (two sources to keep in step); a symlink (a text file on Windows by default, as in decision 13); no harness
on the kit (the commit adding the script broke the size rule unnoticed).

## 19. Incentives, not locks (2026-10-04)

Every check can be bypassed: the harness helps an honest team see drift, size and understanding, it does not police
it. _Rejected:_ CI running the base branch's script, so that a batch cannot loosen its own checks.

## 20. Work in several batches (2026-10-07)

Work that takes more than one batch is planned in `sdd/changes/<slug>.md`: its why, then one section per batch with
the criteria it makes true. A fresh reviewer checks the plan and the human validates it. Each batch moves its criteria
into its module's contract and deletes its section, so the file holds only what is left; empty, it goes with its line
in `sdd/roadmap.md`, which only orders the work. Then a fresh reviewer reads each touched module whole, for a refactor
batch when it pays. A criterion is a plain sentence linked to the tests that prove it, and a link to a missing test or
file is drift. _Rejected:_ the whole plan in `roadmap.md` (it grows with the history and is read at every session);
checkboxes (stored status); planned criteria inside contracts (a contract that is not true yet); a When/Then template
(not how people write specs); a squad of personas (more tokens, no proven gain over one builder and a fresh reviewer).
