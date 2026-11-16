# Card 1 — shedengine-cancelled-state

**Create:**
- `plan:internal/shedengine#StateCancelled` -> `const StateCancelled State = "cancelled"`

**Edit:**
- `internal/shedengine#State.valid`
- `internal/shedengine#TestState_Valid`

**Uses:**
- `internal/shedengine/doc.go`
- `internal/shedengine#StateDone`

**Intent:**
Add `cancelled` to the shed state set, so a finished cancel can record its end in the batten record without reading as `done` (D3).
The dobby's done-run clear (`battencli.ClearDoneRun`) takes only `done` runs and deletes the task branch on origin, so a cancelled run must never read `done`.
`State.valid` accepts the new value; the doc comment on the state block counts seven legal values and says `cancelled` is written only by `lym batten cancel`'s finish, never by a Shed persist.
Grep `internal/` for exhaustive switches over `shedengine.State` (`case shedengine.StateDone`, `StateBlocked`) and give `StateCancelled` the branch a finished, non-landed run takes there, the same as `StateDone` for "drives no further" and never as a landing.
`TestState_Valid` gains a row for `cancelled`.

**ImpactSummary:** one new legal shed state; readers that switch over states treat it as finished, not landed.

**Commit:** 1: Shed states gain cancelled for a finished cancel
