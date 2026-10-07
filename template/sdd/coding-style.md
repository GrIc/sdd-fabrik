# Coding style

How code is written in this project. Each question gets one answer, written here and applied to the whole code base:
never one practice in a file and another elsewhere. Start at GENESIS or ADOPT from the conventions to follow, read in
the team's code, never guessed. When the code raises a question this file does not answer, settle it with the human,
add the rule here and apply it everywhere. Formatters and linters enforce what they can.

## Project rules

| Question | Rule |
|---|---|
| Formatter, linter | to settle |
| Indentation, line width | per language: to settle; markdown prose wraps at 120, never tables, code blocks or URLs |
| Naming, file layout | to settle; names say what they hold or do |
| Comments | rare, they say why: no comment restating the code, no banner, no commented-out code |
| Errors | to settle |
| Diagrams | Mermaid, one per topic |
| Commit messages | what the batch does and, when it is not obvious, why |

## Every line earns its place

Each line of code, config, test or doc can be defended in one sentence: what it does here, and what breaks or gets
worse without it. Otherwise it goes.

- No restated defaults, and a default is the framework's, not the generator's: check before cutting.
- Generator output is a draft. A dependency, file or setting arrives with the first code that needs it.
- No validation, error handling or fallback "just in case". Validate at real boundaries; missing required config fails
  at startup.
- Pareto: a heuristic covers 95 to 98% of real cases. For the rest, rethink the approach instead of piling up
  exceptions. Security, data integrity and explicit requirements are never in the remaining few percent.

Before proposing a batch, its author, human or agent, reads the whole diff against these rules and deletes what fails
them.

## Code a human would write

Data decides which cases matter; human logic decides their shape.

- A list follows its natural series. If the data needs "one", "two" and "five", write one to ten: the series a person
  would expect, not every number the data happened to show, nor a stretch to twelve. If it needs Monday and Friday,
  write the whole week. A gap or a stray entry makes the next reader ask "why?".
- A list holds one kind of thing and is named for it: units in `units`, the prefixes that scale them ("kilo", "milli")
  in a list of their own.
- Completing a series is cheap; adding a rule or a branch for a rare case is not.

## Tests

Test as well as possible, with Pareto applied to coverage too. Weigh each test: what it protects against what it
costs, to write and run, and for the human to review. The code that matters (the rules the contract promises, data
integrity, money, security) is always covered well. Beyond it, a test stays while it pays for itself: 100% is fine
when it comes cheap, never a goal in itself.

## Commands

| Check | Command |
|---|---|
| Tests | to fill at GENESIS or ADOPT |
| Linters | to fill at GENESIS or ADOPT |
| Other CI checks | to fill at GENESIS or ADOPT |
| SDD audit | `bash sdd/bin/sdd audit` |
