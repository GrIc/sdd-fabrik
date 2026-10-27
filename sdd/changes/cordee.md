# The rope team

Why: the harness guides and senses the humans as well as the agents, learns from them and adapts, without watching
them. Design, with what was rejected: `docs/superpowers/specs/2026-10-26-cordee-design.md` (local). Each batch writes
its decision to `docs/decisions.md` with a `Rejected:` line, updates `template/sdd/WORKFLOW.md` or `template/AGENTS.md`
where its behavior shows, and moves its criteria into its module's contract, with the test names below.

Formats the batches share:

- Causes, as the hook reads the ticked boxes of a feedback block: `not-understood`, `unclear`, `not-wanted`, joined by
  `,`, or `-`. The free text after `Other:` is `-` when empty. Tabs and newlines in a title or a free text become
  spaces.
- The quiz journal, `$(git rev-parse --git-common-dir)/sdd/quiz-journal`, shared by worktrees, never pushed. One line
  per question, tab-separated, appended at each graded try:
  `v1, <epoch>, <try|pass>, <fingerprint>, <modules joined by "," or ->, <question number>, <tries>, <causes>, <title>,
  <free text>`. `try` lines come from a failed try, `pass` lines from the try that passes. Readers count a flag once
  per module, title and cause.
- Ledger quiz lines, in the same note as the tokens: `q1, <month>, <module>, <tries>, <causes>, <title>, <free text>`,
  tab-separated, one per question and module of the batch. `report` reads `v1` lines as today and ignores the others.

## A harness that rewrites itself (cockpit)

- `cockpit hints` also reads the quiz journal, passed by `sdd/bin/cockpit` as a third argument: a module flagged
  `not-wanted` or `unclear` twice or more, counted once per title and cause, since the start of the dev's last 10
  batches, gives "HINT Quiz answers in <module> were flagged unclear or not what you wanted <n> times: propose the
  rule you would write.", with the same back-off as the other hints:
  `test_hints_name_a_module_whose_quiz_is_flagged_twice`.
- WORKFLOW step 7: the agent puts it to the dev with the rule it would write (contract, coding style or quiz
  instructions), and writes it only on a yes, in a batch of its own.
