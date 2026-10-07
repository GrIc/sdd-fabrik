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
  contract, or a contract with an empty `governs`, stops every commit. `governs` entries are git pathspecs.
- Size: at most `MAX_LINES` changed lines of code and tests, and one module, unless `Large-Batch: <reason>`.
- Quiz: three questions on the staged batch in `QUIZ.md`, graded offline against a key.
- CI replays drift and size on each new commit, never the quiz.
- A link in a contract is drift when the commit lacks the file it points to:
  [test_a_link_to_a_missing_file_is_drift](/test/run).
- A link is drift when its text is neither found, as whole words, in the file it points to, nor that file's name or
  path: [test_a_link_to_a_missing_test_name_is_drift](/test/run).
- A link may name a test or a group, even wrapped over two lines, or point to a whole file when its text is the file's
  name or path. Its target may be relative to the contract or start at the repo root, with or without an anchor. Web
  links and links to markdown or to a directory are not checked:
  [test_audit_accepts_links_to_names_groups_and_files](/test/run).
- Guidance starts with `NOTE` and never fails `audit` or a commit: the size ratio, governs paths that match nothing,
  a quiz older than its batch: [test_audit_prints_the_size_report](/test/run),
  [test_audit_points_out_governs_paths_that_match_nothing](/test/run),
  [test_a_quiz_older_than_the_batch_only_warns](/test/run).

Constraints: bash 3.2, the one macOS ships (no `mapfile`, no associative arrays, no `${var,,}`), and git with POSIX
tools, on Linux, macOS and Windows (Git Bash).
Non-goals: locks, an LLM at commit time, drift below the file level, CI for other forges (they run the same command).

Formats and usage: [WORKFLOW.md](../../template/sdd/WORKFLOW.md). Proof: the test names in [test/run](../../test/run).
