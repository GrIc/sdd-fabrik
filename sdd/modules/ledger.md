---
governs: [template/sdd/bin/ledger, test/ledger]
---
# ledger

What agents spent on each module, by month, so that a team weighs the cost of its product's features, never of its
people. The details stay on each dev's machine, in `sdd-cockpit`; the repo only keeps the sums. Optional: without
`sdd-cockpit`, nothing is noted, and the ledger never stops a commit or a push.

- `record` splits the tokens spent since the last commit over the modules HEAD changes, by their changed lines, into
  a note of the git notes `sdd-ledger` that hangs on a random key, written by `sdd-fabrik` on the first of the month:
  [test_record_splits_a_batch_by_module_on_no_commit](/test/ledger). A batch outside the modules goes to `-`:
  [test_a_batch_outside_the_modules_goes_to_a_dash](/test/ledger).
- Each window starts where the last record ended, so that an amend counts nothing twice and a failed
  `sdd-cockpit` loses nothing:
  [test_an_amend_counts_from_the_last_record](/test/ledger), [test_a_failed_cockpit_loses_no_window](/test/ledger).
- Without `sdd-cockpit`, nothing is noted: [test_record_without_cockpit_notes_nothing](/test/ledger).
- `sync` merges the remote's ledger into ours and pushes it, so that two devs' notes add up:
  [test_sync_merges_two_devs_ledgers](/test/ledger).
- `report` sums the ledger by month, module, role and model, then shows per month the batches noted against the
  agent commits: [test_report_sums_by_month_module_role_and_model_then_shows_coverage](/test/ledger). Without a
  ledger, it says how to fetch it: [test_report_without_a_ledger_says_how_to_fetch_it](/test/ledger).

Constraints: as [checks](checks.md): bash 3.2 and git with POSIX tools.
Non-goals: the readers of each agent's logs (they live in `sdd-cockpit`, so a new tool changes nothing here), prices,
anonymity in a team of two (a module and a month can still name someone).
