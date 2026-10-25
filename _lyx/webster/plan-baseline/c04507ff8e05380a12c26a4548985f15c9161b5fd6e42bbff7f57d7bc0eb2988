# Card 10 — drift-await-producer

**Create:**
- `internal/driftengine#DriftAwait`
- `internal/driftengine#NewDriftAwait`
- `internal/driftengine#DriftAwait.Call`
- `internal/driftengine#TestDriftAwait`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `internal/driftengine#awaitCaptureEnd`
- `internal/driftengine#ReadScope`
- `internal/driftengine#ReadCapture`
- `internal/driftengine#UpdateScope`
- `internal/driftengine#DirRel`

**Intent:**
Add `Drift-Await`, the mechanical row that holds `Webster` until the baseline exists (D5); it spawns nothing.

`DriftAwait.Call` reads the scope record, and the capture record only after a lock probe, so a capture that just ended is read as ended; it commits all of `DirRel()`, the capture log included, through `Commit` on every way out, Done, Stuck and a deferred Done alike (D4):
no scope record, or scope `off`, is Done;
`awaited_at` set, by a passed await or a deferral, is Done whatever HEAD is, so a later entry after `Webster` committed code passes;
a capture holding the lock is waited on through `awaitCaptureEnd`, with no bound of its own, and the record is read once the lock is free; the held lock reads as activity to the loop's idle watchdog (card 12-drift-activity);
no capture record under `partial` or `full`, a `running` record whose lock is free, and a `failed` record are Stuck, naming the log when there is one;
a `passed` record is Done when HEAD equals its `head`, after writing `awaited_at` into the scope record, and Stuck when HEAD moved.
A Stuck while HEAD equals the branch point names `lyx drift capture` followed by `lyx loom resume`, which captures again on the branch point and re-enters this row, and `lyx drift defer` followed by `lyx loom resume`; a Stuck after HEAD moved names only `lyx drift defer` followed by `lyx loom resume`, since no capture may run on changed code.
None of its Stucks carries the decision halt prefix, so the arbiter may settle one with the way forward it names.

`doc.go` gains the `Drift-Await` section, the counted capture and the adopt case: an adopting machine has no `.lyx/drift/`, so before `awaited_at` the row halts on the missing record and `lyx drift capture` captures there.

`TestDriftAwait` (tier0) is a table over fake `Deps`:
no scope and scope `off` are Done; `awaited_at` set is Done with HEAD moved; a missing record under `partial` is Stuck naming `lyx drift capture` and `lyx drift defer`; a `failed` record, a timed-out record whose state is `failed`, and a `running` record with a free lock are Stuck naming the log; a lock held by the test is waited on, through a fake `Sleep` that releases it, and then the `passed` record is Done and `awaited_at` written; a record the test flips from `running` to `passed` as it releases the lock is read as `passed`, since the read follows the probe; a `passed` record with HEAD moved before the first await is Stuck naming only `lyx drift defer`; after `Defer` the row is Done on every later entry; `Commit` runs on every way out.

**ImpactSummary:** Only the new package's doc grows; no existing code changes.
