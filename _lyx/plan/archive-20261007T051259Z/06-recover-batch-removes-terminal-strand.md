# Card 6 — recover-batch-removes-terminal-strand

**Edit:**
- `internal/websterengine#awaitTerminal`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/doc.go`

**Intent:** D5, recover-batch half: `recover-batch` removes its recovery strand on every terminal outcome whose strand is still live, so a finished recovery strand never stays alive beside its successor.

In `awaitTerminal`, call `removeStrand()` for every terminal digest status: done (strand and run dir, as today), stuck (strand, run dir kept, as today), and dead, which today keeps both; dead now removes the strand and still keeps the run dir for diagnosis.
Move the strand removal ahead of `refuseMidMerge` and `reconcileReportHead`, right after the running check, leaving the done digest's run-dir removal where it is, so a terminal digest that then refuses (a merge in progress, a report head mismatch) leaves no live strand either; the batch stays non-terminal and retryable as today, and the next call re-classifies from the report and events on disk.
The failed-batch path (`PersistRecoveryTerminal` failing the batch after a done digest) runs after `awaitTerminal`, so its strand is already gone.
`removeStrandIfLive` already logs each teardown at Warn; a removal failure stays a warning on the result, as today.
Update `awaitTerminal`'s doc comment, which today says "dead keeps both", and the recovery paragraphs of `doc.go`, which say a terminal recovery digest removes its recovery strand.

Bound: only the strand webster's own batch state names as the batch's recovery strand (`bs.StrandGUID`); an implementer fork or Master strand is never touched.

Tests in `recoverbatch_test.go`, through `RecoverAwait`, the call `lyx webster recover-batch` makes after its spawn-or-attach:
`TestRecoverBatch_TimeoutAcrossCallsClassifiesDead` asserts the dead digest now removes the live strand and keeps the run dir; add a row or case where the digest is terminal but `refuseMidMerge` refuses, asserting the error and that the strand was removed.

**ImpactSummary:** One function in websterengine's recover path; a dead or refused-terminal recovery now tears down its own recovery strand, nothing else changes.
