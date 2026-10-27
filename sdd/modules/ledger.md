---
governs: [template/sdd/bin/ledger, test/ledger]
---
# ledger

What agents spent on each module, by month, and the quizzes passed there, so that a team weighs the cost of its
product's features and sees where they are hard to understand, never its people. The details stay on each dev's
machine, in the [cockpit](cockpit.md); the repo only keeps the sums and the quizzes without a name. On by default;
with `git config sdd.ledger false`, nothing is noted, and the ledger never stops a commit or a push.

- `record` splits the tokens spent since the last commit over the modules HEAD changes, by their changed lines, into
  a note of the git notes `sdd-ledger` that hangs on a random key, written by `sdd-fabrik` on the first of the month:
  [test_record_splits_a_batch_by_module_on_no_commit](/test/ledger). A batch outside the modules goes to `-`:
  [test_a_batch_outside_the_modules_goes_to_a_dash](/test/ledger).
- The same note takes the quiz HEAD passed, from the [quiz journal](checks.md): one line per question and module, with
  the tries, the causes ticked, the title and the dev's words, never the batch or who passed it. The journal's lines
  are matched by HEAD's fingerprint, since worktrees share the journal. A batch with a quiz and no tokens still gets
  its note: [test_record_adds_the_batch_quiz_to_its_note](/test/ledger),
  [test_a_quiz_without_tokens_is_noted](/test/ledger).
- Each window starts where the last record ended, so that an amend counts nothing twice and a failed cockpit loses
  nothing:
  [test_an_amend_counts_from_the_last_record](/test/ledger), [test_a_failed_cockpit_loses_no_window](/test/ledger).
- For a dev who turns it off, nothing is noted: [test_a_dev_who_turns_it_off_notes_nothing](/test/ledger). The hooks
  `install-hook` writes run `record` at each commit and `sync` at each push:
  [test_installed_hooks_record_and_share_the_ledger](/test/ledger).
- `sync` merges the remote's ledger into ours and pushes it, so that two devs' notes add up:
  [test_sync_merges_two_devs_ledgers](/test/ledger). A dev with no ledger fetches nothing:
  [test_sync_without_a_ledger_fetches_nothing](/test/ledger).
- `report` sums the ledger by month, module, role and model, then shows per month the batches noted against the
  agent commits, marked as the hook marks them:
  [test_report_sums_by_month_module_role_and_model_then_shows_coverage](/test/ledger). Without a ledger, it says how
  to fetch it: [test_report_without_a_ledger_says_how_to_fetch_it](/test/ledger).
- `report` then shows the quizzes by month and module (questions, right at the first try, flagged unclear, flagged not
  what the dev wanted), and per module how many devs committed a passed quiz there, a number and never a name:
  [test_report_shows_the_quiz_by_month_and_module](/test/ledger),
  [test_report_counts_who_passed_a_quiz_per_module](/test/ledger).
- `quiz <module>` lists the module's questions of the last three months that were flagged unclear or commented, one
  per line, for the reviewer to read before writing a quiz:
  [test_quiz_lists_a_module_s_unclear_questions](/test/ledger).

Constraints: as [checks](checks.md): bash 3.2 and git with POSIX tools, LF line ends even on a Windows checkout:
[test_ledger_survives_a_crlf_checkout](/test/ledger).
Non-goals: the readers of each agent's logs (they live in the cockpit, so a new tool changes nothing here), prices,
anonymity when one person alone worked on a module in a month (the module and the month name them).
