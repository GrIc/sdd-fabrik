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

## Quiz journal and the quiz feedback call (checks)

- Each graded try appends its lines to the quiz journal, in the format above, the modules being those whose governed
  paths or contract the batch changes: `test_each_try_appends_to_the_quiz_journal`. Worktrees of a clone share it:
  `test_worktrees_share_the_quiz_journal`.
- `AGENTS.md` kit block: "quiz feedback" from the dev means read `QUIZ.md` and the journal's lines for this batch's
  fingerprint, talk the questions through, propose a fix to the batch and, when the question or a guide is at fault, a
  change to the harness, made only on a yes.

## Quiz by risk (checks)

- A contract may declare `risk: high` in its front matter. A batch that changes the governed paths or contract of such
  a module needs five questions, and `audit` notes that a second human's reading is recommended. Otherwise three:
  `test_a_high_risk_module_asks_five_questions`.
- A batch whose changed files are all Markdown outside `sdd/modules/` needs no quiz; a `QUIZ.md` present is still
  graded: `test_a_markdown_only_batch_needs_no_quiz`.
- The trailer records the count: `Quiz: 3/3` or `Quiz: 5/5`. Existing quiz tests keep passing.
- WORKFLOW: `risk: high` in the contract format, the levels in the quiz section, the second reader as a recommendation
  in the handoff (step 7). A dev alone uses the whole harness.

## Quiz answers in the ledger (ledger)

- `record` adds to the batch's note one `q1` line per `pass` line of the journal in its window, for each module of that
  line, with the commit's month: `test_record_adds_the_batch_quiz_to_its_note`. A batch with a quiz and no tokens
  still gets its note: `test_a_quiz_without_tokens_is_noted`.
- With `sdd.ledger false`, no quiz line is noted either: `test_a_dev_who_turns_it_off_notes_nothing` (extended).

## The map of understanding (ledger)

- `report` adds a section by month and module: questions, right at the first try, flagged unclear, flagged not what I
  wanted: `test_report_shows_the_quiz_by_month_and_module`.
- Then per module, how many distinct authors committed a `Quiz:` trailer touching it (through `sdd touched`): a
  number, never a name: `test_report_counts_who_passed_a_quiz_per_module`.
- `ledger quiz <module>` prints that module's questions flagged unclear and the free texts, of the last three months,
  one line each: `test_quiz_lists_a_module_s_unclear_questions`. WORKFLOW step 6: the reviewer runs it before writing
  a quiz. `guide/tokens.md` says what the new section shows.

## Belay: a dev's first batch in a module (checks)

- `sdd brief <module>` prints "First batch of yours in <module>." when the clone's `user.email` authored no commit
  touching the module's contract or governed paths, then who delivered there, the three most frequent authors by name;
  nothing otherwise: `test_brief_spots_a_dev_s_first_batch_in_a_module`, `test_brief_is_silent_in_a_known_module`.
- WORKFLOW step 1: the agent runs it and, on that line, offers discovery mode (sum up the contract and its decisions,
  ask the dev how they would go about it, keep the batch small, ask why rather than which line, suggest pairing with
  someone named), applied only on a yes. Quiz results are never used.

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
