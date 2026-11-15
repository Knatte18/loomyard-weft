# Card 2 — shedrun-intent-writers

**Create:**
- `internal/shedrun#WriteCancelIntent`
- `internal/shedrun#FinishCancel`
- `internal/shedrun#WritePauseIntent`
- `internal/shedrun#ClearPauseIntent`
- `internal/shedrun#TestRunIntents`

**Extends:**
- `internal/shedrun/doc.go`

**Uses:**
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`

**Intent:** The writers of the batten record's intents, under shedrun's tracked paths, as the record's Exports name them.
Each rewrites `StatusFile(l, runID)` under `StatusLock(l, runID)` through `state.UpdateJSON`, so keys the binary does not know survive (`PATTERN-tolerant-records`).
Each writes nothing and returns found false when the status file is absent: a run with no record has no intent to hold.
Each creates the status lock's directory first, as the batten verbs do, since a record pulled from origin has no `.lym` beside it.
None commits or pushes; the verb that calls it commits the record with the run's other records and spawns today's push.

- `WriteCancelIntent` sets `Cancel` to intent, replacing any intent recorded before, its finish included.
- `FinishCancel` sets `Cancel.Finish` to finish and the state to `shedengine.StateCancelled`.
  With no `Cancel` recorded it writes nothing and returns an error naming the missing intent: a fail-loud wiring guard, since the cancel writes its intent before it ends anything (card 7).
- `WritePauseIntent` sets `PauseIntent`; `ClearPauseIntent` sets it nil and reports found true even when none stood.

`TestRunIntents` is tier0: a table over a `lymcwd.Location` in `t.TempDir()`.
Rows: each writer over an absent file writes nothing and reports false;
a cancel intent then its finish leaves state `cancelled` with both recorded;
an intent written over a finished one replaces it whole;
a pause intent written then cleared;
an unknown top-level key in the file survives every writer.

`doc.go` gains a sentence that the intent writers own the batten record's `cancel` and `pause_intent` keys.

**ImpactSummary:** New functions only, beside the existing status path constructors; no caller changes on this card.
