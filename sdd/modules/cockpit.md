---
governs: [template/sdd/bin/cockpit, template/sdd/bin/cockpit.py, test/cockpit]
---
# cockpit

The dev's own view of where their agent tokens go, on their machine. It ships with the harness, and an upgrade
replaces it without an agent reading its diff, so that supporting a new tool costs agents nothing. The only thing it
gives away is `usage`, the sums the [ledger](ledger.md) asks for.

- `usage <repo> <since> <until>` sums the tokens spent in a repo over a window by role and model family (`opus` for
  `claude-opus-5-5`), with output including reasoning, for each tool below.
- Claude Code, read with bash and awk alone: each reply counts once, a half-written last line, a sub-agent's result and
  what a tool call's input holds are no reply; the session is the writer, a sub-agent whose description names a review
  or a quiz is the reviewer: [test_claude_code_splits_writer_and_reviewer_once_per_reply](/test/cockpit). Without
  Python, it says nothing more: [test_claude_code_needs_no_python](/test/cockpit).
- opencode, read with Python 3 since its logs are a SQLite base: a child session whose title names a review is the
  reviewer: [test_opencode_child_session_is_the_reviewer](/test/cockpit).
- `report <repo> [<since> <until>]`, with Python 3, over the last 7 days by default, shows Claude Code's cost in
  input-equivalent tokens (Anthropic's price ratios) by role and model, the context a session and a sub-agent start
  with, the longest context and the share of cache reads: all exact. Then, estimated from their size, what is sent along
  with the calls (tool results, skill listing, hooks, MCP instructions, memory files), each written once and read again
  with every later call: [test_report_weighs_roles_and_what_is_sent_along](/test/cockpit).
- `hints`, with Python 3, prints what the last 7 days show to cut: MCP servers whose instructions are sent but whose
  tools were never called, plugins never called when the skill listing weighs 5% of the cost or more, anything else
  sent along at 5% or more, a context over 150k tokens. Each comes back a week later at the earliest; without Python,
  it says nothing: [test_hints_say_what_to_cut_once_a_week](/test/cockpit),
  [test_claude_code_needs_no_python](/test/cockpit).

Constraints: `usage` for Claude Code as [checks](checks.md) (bash 3.2, POSIX awk); the rest in one Python 3 file,
standard library only; tested on Linux and macOS.
Non-goals: a report for opencode (only its sums for now), tools whose logs we have not read yet (Codex), telling
apart two batches worked on at the same time, changing the dev's setup (the agent asks, the dev decides).
