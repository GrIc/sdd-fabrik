# Developing SDD-Fabrik

- `template/` is what lands in user repos. Edit it; never copy it elsewhere in the kit.
- The kit follows its own rules: `template/AGENTS.md` and `template/sdd/coding-style.md`.
- `template/sdd/bin/sdd` stays bash 3.2 compatible (macOS): no `mapfile`, no associative arrays, no `${var,,}`. Only
  git, sed, awk and grep.
- Run `bash test/run` before proposing a batch.
- Use a fresh reviewer, a new context, for the final diff of a change.
- `docs/feedback/` is local input from field use: read it, never commit it.
- Decisions about the kit go to `docs/decisions.md`, with a `Rejected:` line.
- Never commit, push or tag: the human does.
