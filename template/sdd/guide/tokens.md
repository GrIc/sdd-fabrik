# Tokens: what agents spend

`sdd/bin/ledger` keeps what agents spent on each module, by month, in the git notes `sdd-ledger`: the cost of the
product's features, with no commit, author or date. The hooks `install-hook` writes note it at each commit and share it
at each push, unless the dev sets `git config sdd.ledger false` (`--global` for every repo). The tokens come from
`sdd/bin/cockpit`, which reads the agents' logs on this machine: Claude Code's with bash and awk, opencode's with
Python 3. `bash sdd/bin/cockpit report "$PWD"`, from the repo root and with Python 3, shows a dev where their own
tokens went; `bash sdd/bin/cockpit hints` says what to cut, once in 10 batches at most. To audit a clone,
`git fetch origin refs/notes/sdd-ledger:refs/notes/sdd-ledger`, then `bash sdd/bin/ledger report`: the tokens by month,
module, role and model, then per month the batches noted against the agent commits.

The same notes keep each quiz passed, one tab-separated line per question and module: `q1`, the month, the module, the
tries, the boxes ticked, the question's title and the dev's own words, as in the [quiz journal](../WORKFLOW.md#quiz).
They are pushed with the tokens: the dev's words reach the team, never their name. `report` then shows them by month and
module: the questions, those right at the first try, those flagged unclear or not what the dev wanted. A module where
few are right at the first try is hard to understand, in its code or in its quizzes. Last, per module, how many devs
passed a quiz there: one alone means nobody else could take over.
