# Developing SDD-Fabrik

- `template/` is what lands in user repos. Edit it; never copy it elsewhere in the kit.
- The kit follows its own rules and runs its own harness: `template/AGENTS.md`, `template/sdd/WORKFLOW.md` and
  `template/sdd/coding-style.md` apply here, `sdd/bin/sdd` runs the template's script, and its contract is
  `sdd/modules/checks.md`.
- Run `bash test/run` before proposing a batch.
- `docs/feedback/` is local input from field use: read it, never commit it.
- Decisions about the kit go to `docs/decisions.md`, with a `Rejected:` line.
- Never commit, push or tag: the human does.
