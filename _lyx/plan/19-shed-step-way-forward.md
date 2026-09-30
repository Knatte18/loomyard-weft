# Card 19 — shed-step-way-forward

**Edit:**
- `internal/shedengine/run.go`
- `internal/loomengine/anomaly.go`
- `internal/shedengine/goto_test.go`
- `internal/loomengine/anomaly_test.go`

**Intent:**
Name `goto` as the way forward from the two shed stops that had none (D11): a `current_producer` a deploy renamed or removed, and an exhausted bounce budget that re-blocks on every re-step.

`run.go`:

- Step 2's lookup refusal keeps its diagnosis and ends `way forward: "lyx shed goto --to <producer>" moves the run onto a row that exists; valid producers: <every name in list order>`.
  It still changes nothing on disk.
- The budget-exhausted arm persists `ReasonBounceBudgetExhausted` followed by `for <row>; way forward: "lyx shed goto --to <row>" gives <segment "S" | row "R"> a fresh budget`.
  `ReasonBounceBudgetExhausted` stays the reason's exact prefix; its doc comment says so.

`anomaly.go`: `detectHaltAnomaly` recognises budget exhaustion with `strings.HasPrefix(final.Error, shedengine.ReasonBounceBudgetExhausted)`, since the persisted reason now carries a suffix.
Update the partition comment.

Tests:

- `goto_test.go` (shedengine): a status file whose `current_producer` names no producer → `Step` refuses with a message containing `lyx shed goto` and every valid name; `Goto` to a real row leaves the run `paused` there; the next `Step` calls that producer.
  A segment whose row is blocked on budget exhaustion → the persisted error names `lyx shed goto`; after `Goto` to that row, the next Stuck bounces rather than blocking.
- `anomaly_test.go`: a blocked final status whose error is the prefixed reason still maps to the budget-exhausted anomaly kind, and a blocked status with another reason still maps to escalation.

**ImpactSummary:** Two refusal texts gain a way forward; loom's anomaly classifier is the one reader of the persisted reason and moves to a prefix match.
