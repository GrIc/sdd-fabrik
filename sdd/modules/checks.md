---
governs: [template/sdd/bin/sdd, template/.github/workflows/sdd.yml, test/run]
---
# checks

The script a deployed project runs at each commit and in CI. It keeps each contract in step with its code, keeps agent
batches small, and makes the human understand an agent batch before it is committed. It helps, it never locks: every
check can be bypassed.

- An agent batch (`Agent-Assisted: yes`, or a co-author address in `AGENT_EMAILS`) is refused on drift, on size or on
  a wrong quiz. The human's own commits only get warnings on drift and size: [test_agent_drift_blocks](/test/run),
  [test_an_agent_co_author_marks_an_agent_batch](/test/run), [test_human_drift_warns_and_commits](/test/run),
  [test_over_max_lines_blocks_an_agent_warns_a_human](/test/run).
- Drift: a commit that changes a module's governed paths changes its contract too, or names the module in
  `Spec-Unchanged(<module>): <reason>`, which also leaves it out of the one-module limit. A `Spec-Unchanged` naming no
  contract, or a contract with an empty `governs`, stops every commit. `governs` entries are git pathspecs:
  [test_spec_unchanged_spares_its_module_only](/test/run),
  [test_a_spared_module_does_not_count_toward_the_module_limit](/test/run),
  [test_errors_stop_even_a_human_commit](/test/run), [test_governs_takes_git_pathspecs](/test/run).
- Size: at most `MAX_LINES` changed lines of code and tests, and one module, unless `Large-Batch: <reason>`:
  [test_over_max_lines_blocks_an_agent_warns_a_human](/test/run), [test_two_modules_block_an_agent](/test/run),
  [test_large_batch_lifts_both_limits](/test/run).
- Quiz: three questions on the staged batch in `QUIZ.md`, graded offline against a key:
  [test_quiz_passed_adds_the_trailer_and_cleans_up](/test/run),
  [test_wrong_answer_explains_then_a_retry_passes](/test/run).
- CI replays drift and size on each new commit, never the quiz: [test_range_checks_each_commit](/test/run).
- A link in a contract is drift when the commit lacks the file it points to, or when its text is neither found, as
  whole words, in that file, nor its name or path: [test_a_link_to_a_missing_file_is_drift](/test/run),
  [test_a_link_to_a_missing_test_name_is_drift](/test/run).
- A link may name a test or a group, even wrapped over two lines, or point to a whole file when its text is the file's
  name or path. Its target may be relative to the contract or start at the repo root, with or without an anchor. Web
  links and links to markdown or to a directory are not checked:
  [test_audit_accepts_links_to_names_groups_and_files](/test/run).
- Guidance starts with `NOTE` and never fails `audit` or a commit: the markdown ratio, governs paths that match nothing,
  a quiz older than its batch: [test_audit_notes_the_markdown_ratio](/test/run),
  [test_audit_points_out_governs_paths_that_match_nothing](/test/run),
  [test_a_quiz_older_than_the_batch_only_warns](/test/run).

Constraints: bash 3.2, the one macOS ships (no `mapfile`, no associative arrays, no `${var,,}`), and git with POSIX
tools, on Linux, macOS and Windows (Git Bash).
Non-goals: locks, an LLM at commit time, drift below the file level, CI for other forges (they run the same command).

Formats and usage: [WORKFLOW.md](../../template/sdd/WORKFLOW.md).
