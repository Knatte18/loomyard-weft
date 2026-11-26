---
round: 1
focus:
  - "D2, D3 and D13 together: for each detached child (teardown child, pair push, background pusher's push), how does it get the seam-supplied facts, run paths, run locks and verdicts when no in-process caller can hand it a seam, and does the record decide that?"
  - "D2: check against the code that phased snapshots can keep both bounds, no verb starting more git processes and no verb reading more before a refusal. Check the premises about `gitrepo`'s Go reads and `dotgit.ListWorktrees`, including the pin for each tool behaviour the record rests on."
  - "D3 and D4: is every mid-flow decision, re-read, live probe, lock hand-off and failure path of today's verbs expressible as an action list, a phase or an execution-time guard? Sample the teardown, merge and landing, push rejection and `Add` rollback flows, including a failed or killed action and an interrupted rollback."
  - "D3, D13 and the attack-surface question: does each new seam, action kind and moved responsibility state what it can skip or let through and what bounds it, including a seam the caller leaves unset or answers wrongly?"
  - "D13: check the premise against fabric's production imports. Are the four cut modules and `internal/configengine` the whole set of imports that sit above fabric under the decision's own definition, and can each cut use (run status, takeover, run paths and locks, stencil ordering, workspace build) be replaced without changing behaviour (D5), also where `internal/fabriccli` or a child process is the caller?"
  - "D1 and D7: can `internal/fabriclayout` and `internal/fabriccore` hold what the record assigns them under a closed standard-library-and-leaf import set? Check the types and path functions that would move for dependencies on non-leaf packages, and check the gate-scope rationale against `internal/impactset`."
  - "D12 and D6: can fabric's tests drop every import of a module above fabric while still using `internal/hubforge`, real config and real seam data? Is there a kind of existing test that neither the D6 sorting nor the D12 sorting places, or whose assertion would be lost?"
  - "D5 and D9: does the part order let each part delete or move tests only once their replacements exist, and keep unmoved concerns working beside the new core? Look for cross-concern dependencies that cut against the stated order, such as push, merge and teardown sharing code."
  - "D10: what evidence does the 60 s target and the whole-run measure rest on, and can the close-out part measure them against the board entry's baseline as written?"
  - "Board fit: read this task's board entry, its `Decision for` notes and the touching entries (`push-retry-not-halt`, `hardener-study-faults`, `keep-landed-run-records`). Does the record cover the entry's scope without exceeding or contradicting it, and which `## Open risks` lines are questions only the run's parent can settle?"
  - "`## PATTERN changes` and `## Constraints`: compare each bullet with the current PATTERN entry and its background. Does a rewritten rule line drop a clause still needed, agree with the Notes on `PATTERN-lym-files`, and match D1, D4, D7, D12 and D13?"
---

Seed pass: no round has been reviewed.
The entries aim at the design's premises and at the places where two decisions meet.
