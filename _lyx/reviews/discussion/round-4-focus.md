---
round: 4
focus:
  - "Constraints in `_lyx/discussion/decision-record.md`: D13 to D15 put the weft and warp tokens into `internal/planparser` and its test fixtures, the `card-fabric-reference` row in `loom-plan-spec.md`, the committed-data steer in `loom-template-plan.md` and `websterengine` tests. Settle whether Constraints state the `PATTERN-fabric-vocabulary` rule (a scan outside the owner set fails on the bare tokens, with only `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push` exempt) so the plan writer can write those files."
  - "D13 and the Goal in `_lyx/discussion/decision-record.md`: D13 scans the fabric-driving spelling only in a `**Verify:**` value, the `## verify:` section and fenced blocks, and passes an inline code span and prose. Settle whether the Goal's guarantee sentence and D13's Bound state exactly that scope, and whether the Bound's wording about absolute paths and `..` escapes agrees with the sibling-name rule that refuses a relative weft path."
  - "D13's claim that the audit's matches stay unchanged: the audit's exact-path alternative in `internal/fabricengine/refscanner.go` (`NewRefScanner`) has no name-end anchor. Settle whether the record's description of what is shared with the audit is true of that alternative."
  - "D11's profile line in `_lyx/discussion/decision-record.md`: findings of multi-profile and restarted runs enter no profile line while their cards enter the line's `cards`. Settle whether the denominator of the line's Webster-Review findings per card is defined."
---

Focus targets the new text from the round-3 fixes and the one gating question; the rubric's `Do not flag` list and writer-symmetry rule were checked against each entry.
