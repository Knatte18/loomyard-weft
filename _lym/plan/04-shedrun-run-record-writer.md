# Card 4 — shedrun-run-record-writer

**Create:**
- `plan:internal/shedrun#CancelLock` -> `func CancelLock(l *lymcwd.Location, runID string) string`
- `plan:internal/shedrun#UpdateRunRecord` -> `func UpdateRunRecord(l *lymcwd.Location, runID string, mutate func(st *shedengine.Status) error) (found bool, err error)`
- `plan:internal/shedrun#TestUpdateRunRecord` -> `func TestUpdateRunRecord(t *testing.T)`

**Edit:**
- `internal/shedrun#TestPathConstructors`

**Uses:**
- `internal/shedrun/doc.go`
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`
- `internal/shedrun#ScratchRoot`
- `internal/shedengine#Status`

**ImpactSummary:** `shedrun` gains one writer for a run's tracked status record and a name for the cancel's local lock; no existing caller changes.

**Intent:**
Give the run's tracked intents one writer under shedrun's tracked paths, and name the cancel's local lock.

`UpdateRunRecord` applies `mutate` to the run's status file at `StatusFile` under `StatusLock`, through `state.UpdateJSON`, creating the lock's never-tracked directory first.
An absent record writes nothing and returns found false; a `mutate` error writes nothing and is returned.
A key the record holds that `Status` does not declare survives the write, as `state` keeps it.
It commits nothing: its callers commit and push the record.
`shedrun` gains the `shedengine` import for it; `shedengine` imports nothing of `shedrun`, so no cycle forms.

`CancelLock` names the lock whose holder is a live `lym batten cancel` of the slug: `<run segment>.cancel.lock` directly under `ScratchRoot`.
It is the same file `CancelMarkerLock` names today, so a cancel run by an older build still holds the lock a newer one probes.
The lock stays never-tracked: a cancel's liveness is local, its intent is tracked.

`TestUpdateRunRecord` covers a write over a present record, an absent record writing nothing, a refusing `mutate` writing nothing, and an unknown key surviving; it is tier 0.
`TestPathConstructors` gains the `CancelLock` row.

**Commit:** 4: shedrun: one writer for a run's tracked record and the cancel's local lock
