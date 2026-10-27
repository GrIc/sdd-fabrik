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

## Belay: the rope team's pace (checks)

- `sdd brief` also prints, at most three lines, nothing when nothing waits: other worktrees of the clone holding a
  `QUIZ.md`; remote branches not merged into `origin/HEAD`, whose last commit is older than `REVIEW_WAIT_HOURS=4` (top
  of the script) and not by this dev, oldest first: "<branch> waits for a review for <n> hours|days, see <author>."
  Without `origin/HEAD`, no branch line. It never fetches: `test_brief_shows_quizzes_waiting_in_other_worktrees`,
  `test_brief_shows_branches_waiting_for_review_with_their_author`, `test_brief_says_three_lines_at_most`.
- WORKFLOW step 1: the agent proposes to finish what waits before producing more.
- Decision: name to coordinate, never to measure.

## A harness that rewrites itself (cockpit)

- `cockpit hints` also reads the quiz journal, passed by `sdd/bin/cockpit` as a third argument: a module flagged
  `not-wanted` or `unclear` twice or more, counted once per title and cause, since the start of the dev's last 10
  batches, gives "HINT Quiz answers in <module> were flagged unclear or not what you wanted <n> times: propose the
  rule you would write.", with the same back-off as the other hints:
  `test_hints_name_a_module_whose_quiz_is_flagged_twice`.
- WORKFLOW step 7: the agent puts it to the dev with the rule it would write (contract, coding style or quiz
  instructions), and writes it only on a yes, in a batch of its own.
