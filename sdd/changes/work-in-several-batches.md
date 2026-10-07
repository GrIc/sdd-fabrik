# Work in several batches

Why: work too big for one batch needs a plan the human validates, criteria that move into contracts as the batches
land, and links that keep each criterion tied to its tests. Decision 20 will record the reasons.

## Plan, criteria and module review in the workflow (no module)

- WORKFLOW's "Where things live" lists `sdd/changes/<slug>.md`; `sdd/roadmap.md` keeps the order of the work, one line
  and link per change, and "Someday"; acceptance lives in contracts, proved by the tests they link.
- AGENTS.md's house rule says the order of the work lives only in `sdd/roadmap.md`; decision 16 is corrected to match.
- A PLAN section: work of more than one batch gets a change file, its why, then one `## <title> (<module>)` section
  per batch with its criteria. An issue taken up moves into the why and is deleted. A fresh reviewer checks that each
  batch fits the size limit, touches one module and has testable criteria; the human validates; the plan is committed
  with its first batch.
- COMPILE: a batch's commit moves its criteria into its module's contract, in place of what they restate, and deletes
  its section; a batch without a module drops them, since the files it changed now say it.
- The first section left is the next batch. The batch that empties a change file deletes it and its roadmap line;
  then a fresh reviewer reads each module the change touched as a whole and proposes a refactor batch only when it
  pays.
- The fallback is said once for the plan, batch and module reviews: a tool that cannot spawn a fresh reviewer does
  its part itself and says so.
- "Contracts" shows the criterion format: a plain sentence, then links to the tests that prove it (a name, a group or
  a whole file; several criteria may share one test; no link means a check by hand that the sentence describes).
- The output of `sdd/bin/sdd` is described once: ERROR stops every commit, DRIFT and SIZE refuse an agent batch and
  warn on the human's own, NOTE is guidance and never refuses. DRIFT also covers a contract link to a missing test or
  file.
- GENESIS: "The first delivery follows PLAN or COMPILE."
- coding-style.md's Tests: a test proves a behavior through the public interface; one test may cover several criteria;
  no micro test for each micro change.
- sdd/modules/checks.md follows the format: each criterion links the tests that prove it, and its "Proof:" line goes.
- README's "How a batch flows" starts with the plan for work of several batches.
- Decision 20 records the model and what was rejected: everything in roadmap.md, checkboxes, planned criteria inside
  contracts, a When/Then template, a full squad of agents.
- This batch empties the change: it deletes this file and sdd/roadmap.md, left with nothing to order.
