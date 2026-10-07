---
governs: [template/sdd/bin/sdd, template/.github/workflows/sdd.yml, test/run]
---
# checks

The script a deployed project runs at each commit and in CI. It keeps each contract in step with its code, keeps agent
batches small, and makes the human understand an agent batch before it is committed. It helps, it never locks: every
check can be bypassed.

- An agent batch (`Agent-Assisted: yes`, or a co-author address in `AGENT_EMAILS`) is refused on drift, on size or on
  a wrong quiz. The human's own commits only get warnings on drift and size.
- Drift: a commit that changes a module's governed paths changes its contract too, or names the module in
  `Spec-Unchanged(<module>): <reason>`, which also leaves it out of the one-module limit. A `Spec-Unchanged` naming no
  contract, or a contract with an empty `governs`, stops every commit. `governs` entries are git pathspecs, and `audit`
  points out the ones that match no tracked file.
- Size: at most `MAX_LINES` changed lines of code and tests, and one module, unless `Large-Batch: <reason>`.
- Quiz: three questions on the staged batch in `QUIZ.md`, graded offline against a key. A batch changed since the
  quiz was written gets a warning, not a refusal.
- CI replays drift and size on each new commit, never the quiz.

Constraints: bash 3.2, the one macOS ships (no `mapfile`, no associative arrays, no `${var,,}`), and git with POSIX
tools, on Linux, macOS and Windows (Git Bash).
Non-goals: locks, an LLM at commit time, drift below the file level, CI for other forges (they run the same command).

Formats and usage: [WORKFLOW.md](../../template/sdd/WORKFLOW.md). Proof: the test names in [test/run](../../test/run).
