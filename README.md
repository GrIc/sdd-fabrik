# SDD-Fabrik

A small harness for building software with coding agents. Intent lives in short markdown contracts next to the code.
A git hook and a CI step catch code that drifts from its contract, batches too big to review, and agent work the
human has not understood yet. Everything is an incentive: any check can be bypassed, on purpose.

Needs only git (on Windows, Git for Windows, which brings bash, sed and awk). Works on Linux, macOS and Windows.

## Deploy (for agents)

You were asked to deploy SDD-Fabrik in the current repository. Do this:

1. Check the dependencies: `git`, `bash`, `sed` and `awk`. If any is missing, tell the human which ones and how to
   install them for their system, and stop.
2. Fetch the latest release into a temporary directory, taking the highest `vX.Y.Z` from the first command:

   ```
   git ls-remote --tags --refs https://github.com/GrIc/sdd-fabrik
   git clone --depth 1 --branch <tag> https://github.com/GrIc/sdd-fabrik <tmp>
   ```
3. Copy `<tmp>/template/` into the repository root without overwriting anything. Where a file already exists:
   - `AGENTS.md`: append the block between `<!-- sdd-fabrik -->` and `<!-- /sdd-fabrik -->`;
   - `CLAUDE.md`: add a line `@AGENTS.md`;
   - `.gitattributes`: append its line, which keeps the script's line ends LF on Windows;
   - a CI workflow: add its step and `fetch-depth: 0` from `.github/workflows/sdd.yml` to it, or keep both;
   - anything else: stop and ask the human.
4. Replace `<tag>` on the first line of `sdd/README.md` with the tag you deployed.
5. Run `bash sdd/bin/sdd install-hook`. If it declines because another hook is in place, show the human the line it
   printed.
6. If your tool has a permission deny list, add `git commit`, `git push` and `--no-verify` to it. If it commits on its
   own (Aider does by default, without hooks), turn that off. Tell the human that for their other agents these rules
   are instructions only.
7. Delete `<tmp>`. Report the files added and merged, the dependencies, and a proposed commit message with
   `Large-Batch: deploys SDD-Fabrik` (the script alone is over the size limit) and without `Agent-Assisted` or an
   agent's `Co-Authored-By` (the files come from the kit, not from you). The human commits.
8. Then follow GENESIS (no code yet) or ADOPT (existing code) in `sdd/WORKFLOW.md`, as your first batch.

To upgrade or uninstall, follow the last two sections of `sdd/WORKFLOW.md` in the deployed repository.

## How a batch flows

1. For work of several batches, the agent first writes a plan the human validates: `sdd/changes/<slug>.md`.
2. The agent builds the smallest coherent change, updates its module contract or says why it stays, and runs the
   checks.
3. A fresh reviewer, a new agent context, reads the staged diff and writes `QUIZ.md`: three questions on the change.
4. The human reads the diff, ticks the answers, and commits, from a terminal or an IDE.
5. For an agent-assisted commit, the `commit-msg` hook checks that each contract moved with its code, that the batch
   is small, and that the quiz is right. For the human's own commits, it only warns.
6. CI replays drift and size on every pushed commit.

## Why

The decisions behind this design, with the alternatives we rejected, are in [docs/decisions.md](docs/decisions.md).

## Develop the kit

Edit `template/`, run `bash test/run`. See [AGENTS.md](AGENTS.md).

MIT licensed, see [LICENSE](LICENSE).
