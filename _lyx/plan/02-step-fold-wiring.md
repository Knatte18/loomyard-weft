# Card 2 — step-fold-wiring

**Edit:**
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.persistTransient`
- `internal/shedengine#Shed.persist`

**Create:**
- `internal/shedengine/step_fold_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/shedengine#appendOrFold`
- `internal/shedengine#scriptedStuckShed`
- `internal/shedengine#funcProducer`

**Intent:**
Route every verdict arm's history write through `appendOrFold`, so a budget-exempt Stuck folds into the preceding identical one inside the step's single persist (D1, D2), and make `activity.last` recompose after a fold (D6).

In `stepLocked`, the `appendHistory` closure keeps its `outcome == ""` skip and its entry construction, but hands the built `HistoryEntry` to `appendOrFold(st.History, entry)` instead of appending to its own copy.
Every arm that calls `appendHistory` therefore gets folding with no per-arm change, and the result is still committed by that arm's one `persist` call.
Update the comment above the closure ("Appended to a copy of the history read at step 1 ...") to say the entry is appended or folded, still on a copy.
The budget comparison keeps reading `st.History`, never `nextHistory`, and `episodeStuckCount` is untouched (D5).

In `persistTransient`, the carry-forward of the file's existing `activity.last` applies today whenever the write did not lengthen history, which would wrongly treat a fold as a history-less pause/resume write.
Replace the `appended` test with "this write recorded a verdict": history got longer, or both histories are non-empty with equal length and the last entry's `Repeats` differs from the re-read file's last entry.
Only a write that recorded nothing — the pause writes and the step-3b resume write — carries `activity.last` forward;
a fold recomposes it through `composeActivity`, so a folded self-bounce reads "`<producer>` → bounced to `<routedTo>`".
Update `persist`'s doc comment paragraph on `routedTo` accordingly: a persist that appends or folds an entry recomposes `activity.last`, and only one that records no verdict carries it forward.
The same doc comment's batten-seam paragraph still holds (a skipped self-bounce's accumulated history, folded or not, is committed whole by the next transition), so reword it only where it says history "accumulated" in a way that implies one entry per call.

`step_fold_test.go` drives a real `Shed` through `Step`:

- repeated exempt Stucks on a self-bouncing row leave the persisted history at one entry whose `Repeats` is the number of steps after the first, whose `At` is unchanged from the first step, and whose `LastAt` is RFC 3339 UTC and not earlier than `At`; `StepResult.History` equals the persisted history;
- the step that folds overwrites `activity.last`: seed a sentinel `activity.last` into the file between the first and second step (through `state.UpdateJSON` on the same lock path) and assert the folding step leaves `"Wait → bounced to Wait"`;
- a folding step on a running run calls a counting `CommitStatus` exactly once, so the fold travels in the step's single persist;
- an exempt Stuck after a counted one, and a counted one after an exempt one, each lengthen the persisted history;
- a producer scripted exempt Stuck, Awaiting, exempt Stuck ends with three entries, the last carrying no `Repeats`/`LastAt`.

In `docs/overview.md`'s batten `status` paragraph, stop claiming `Run-Shed`'s self-bounce appends one entry per poll while the child awaits approval or waits for its driver.
State instead that the running watch appends one entry per poll, bounded by its 1440-bounce budget, while a budget-exempt wait (an awaiting hand-off, the driver-exit grace) folds consecutive identical polls into one entry carrying `repeats` and `last_at`, so such a wait neither grows the history nor advances `history_length` (D10).
Adjust the same bullet's earlier sentence beginning "Those per-poll history appends" so it still reads true for a fold, which rewrites `status.json` on every poll without lengthening it.
Keep semantic line breaks.

**ImpactSummary:** Every Shed caller's history now folds consecutive identical budget-exempt Stucks; battenshed is the only producer emitting them, and counted Stucks, routing and budget decisions are unchanged.
