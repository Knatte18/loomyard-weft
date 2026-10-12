# Card 7 — parentlock-covered

**Edit:**
- `internal/parentlock#Store`
- `internal/parentlock#Store.TryAcquire`
- `internal/parentlock/doc.go`
- `internal/loomcli#parentLockDeps`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/parentlock#TestTryAcquire_CoveredRunYieldsItsTurnAndKeepsItsTicket`

**Intent:**
Make a covered run yield the parent lock queue (D4): it keeps its ticket and its place, is never granted while covered, and an uncovered run behind it is granted a free record as if it were absent.

`Store` gains `Covered func(slug string) bool`; nil covers nothing, so every store built without it behaves as today.
In `TryAcquire`, after the live-holder check:
- an acquirer whose `Owner.Slug` is covered keeps its ticket and is not granted, unless the record already names its own run (`SameRun`), which is D4's run that held the lock when the entry was written and is re-granted as today;
- the queue head is the oldest live ticket whose slug is not covered, so covered tickets stay on disk and are passed over, and a covered waiter still first in the queue at the resume is granted next.
The record itself and liveness are untouched: a covered holder keeps its record, and only its idle release changes (cards 17 and 18).
`Acquire`'s loop needs no change: a covered waiter's attempts keep reading not granted until the resume, so the pause alone never ends its wait with `ErrPaused`.

`parentLockDeps` in loomcli, which builds the store every lock-taking landing row acquires through, sets `Covered` to a read of `pauserecord.ReadCoverage` at `hubgeom.PauseGeometry(l)`'s record path for the asked slug; a read error covers nothing and logs one Warn.
Every acquirer goes through that store, so the yield holds wherever a grant is decided.

Test: the new test builds a store with a fake `Covered` over a holder and three waiters queued in order `first`, `second`, `third` with `first` covered: after the holder releases, `first` is not granted; `second` is granted ahead of it while `first`'s ticket stays on disk; after `second` releases and `first` is uncovered, `first` is granted ahead of `third`; and a covered acquirer whose run already owns the record is re-granted.
`doc.go`'s Queue section names the covered yield.

**ImpactSummary:** One new Store field read in TryAcquire's grant choice; nil keeps every existing store as today, and loom's landing store is the one that sets it.
