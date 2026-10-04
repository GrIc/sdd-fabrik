# Coding style

Whose conventions this project follows, and the commands that check them. Filled at GENESIS or ADOPT from the
conventions the team already uses; the stack's tools decide formatting.

## Every line earns its place

Each line of code, config, test or doc can be defended in one sentence: what it does here, and what breaks or gets
worse without it. Otherwise it goes.

- No restated defaults, and a default is the framework's, not the generator's: check before cutting.
- Generator output is a draft. A dependency, file or setting arrives with the first code that needs it.
- No validation, error handling or fallback "just in case". Validate at real boundaries; missing required config fails
  at startup.
- Pareto: a heuristic covers 95 to 98% of real cases. For the rest, rethink the approach instead of piling up
  exceptions. Security, data integrity and explicit requirements are never in the remaining few percent.
- Names say what; the rare comment says why. No comment restating the code, no banner, no dead code.

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

## Docs and commits

- Markdown prose up to 120 columns; tables, code blocks and URLs never wrapped. Diagrams in Mermaid, one per topic.
- A commit message says what the batch does and, when it is not obvious, why. No flattering claims: each claim has its
  evidence.

## Commands

| Check | Command |
|---|---|
| Tests | to fill at GENESIS or ADOPT |
| Linters | to fill at GENESIS or ADOPT |
| Other CI checks | to fill at GENESIS or ADOPT |
| SDD audit | `bash sdd/bin/sdd audit` |
