---
governs: [cockpit/, test/cockpit]
---
# cockpit

The dev's own view of where their agent tokens go, on their machine, outside the deployed harness. It reads each
tool's logs, so that supporting a new tool changes nothing in a project's repo nor in what its agents read. The only
thing it gives away is `usage`, the sums the [ledger](ledger.md) asks for.

- `usage <repo> <since> <until>` sums the tokens spent in a repo over a window by role and model family (`opus` for
  `claude-opus-5-5`), with output including reasoning, for each tool below.
- Claude Code: each reply counts once, a half-written last line is skipped; the session is the writer, a sub-agent
  whose description names a review or a quiz is the reviewer:
  [test_claude_code_splits_writer_and_reviewer_once_per_reply](/test/cockpit).
- opencode: a child session whose title names a review is the reviewer:
  [test_opencode_child_session_is_the_reviewer](/test/cockpit).

Constraints: one Python 3 file, standard library only; tested on Linux and macOS.
Non-goals: tools whose logs we have not read yet (Codex), telling apart two batches worked on at the same time.
