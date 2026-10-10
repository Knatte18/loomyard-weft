---
round: 1
focus:
  - "decision-record.md D5 and D4: settle whether the seed-commit signature (message prefix plus paths under the stencils or specs subtree) can match a commit that is not regenerable derived content, and whether the stated Bound covers the case where a dropped seed commit carried a hand-edit or a refresh the next pre-run will not reproduce (D1/D2 gating)."
  - "decision-record.md D2 and D1: settle whether the guard downgrade is bounded on each path that skips a refresh (unstamped mode as the zero value, modified tree, missing ancestry, no recorded revision), and whether a file with no recorded revision lets a production binary overwrite an untouched copy in a case the Decision does not bound."
  - "decision-record.md D8: settle whether 'a probe that fails counts as in flight' and the prime-session exemption are bounded, and whether the skip leaves a state the Open risks and Acceptance criteria account for."
  - "decision-record.md D9: settle whether the one-day expiry states what it newly lets gc delete and what bounds it, with the Decision text and Open risks agreeing."
  - "Consistency between Goal, Scope, Decisions, Constraints, Open risks and Acceptance criteria: settle whether every Decision (D1-D9) has a matching acceptance scenario, whether Scope's 'Out' items contradict a Decision (D5 board-sync conflict rule, D7 warp-binding), and whether D4's stamp-absent retry and D5's exclusion of the hub-wide config commit agree."
  - "decision-record.md is the Plan producer's sole input: settle whether any term, such as 'Part 3', 'Part 4' or 'the issue', is used without being defined inside the record, so a reader with only this file cannot act on it."
  - "Relocation and exclusion: settle whether any Rationale, Constraints entry or Notes item is deliberation narrative that belongs in support-log.md, after checking that no requirement the plan writer needs would be lost."
---

Do-not-flag list and the discussion stencil's no-gather rule checked: no entry asks for exact signatures, `file:line` citations, call-site enumeration or rejected alternatives in `decision-record.md`.
