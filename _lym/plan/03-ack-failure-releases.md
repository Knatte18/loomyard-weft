# Card 3 — ack-failure-releases

**Edit:**
- `internal/creelengine#Reading.Ack`
- `internal/creelengine#journalWriter`
- `internal/creelengine#journalWriter.writeLine`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelengine/doc.go`

**Create:**
- `plan:internal/creelengine#Store.FailJournalWrites` -> `func (s *Store) FailJournalWrites(after int)`

**Intent:**
Decision D3's ordering rule: a journal write that fails at an acknowledgement releases every id the acknowledgement left unjournaled, where today `Reading.Ack` returns at the failing id with that id and the ones after it still in `Store.reserved`, so no later claim or read hands them out until the daemon restarts.

`Reading.Ack`: when `journalReadLocked` fails for an id, Ack deletes the reservation of that id and of every id after it, answers the holder's mail waiters as `Reading.Release` does when it released any, clears `r.ids` and returns the error.
The ids before the failing one stay journaled read.
The change covers every reading, a `read`'s included, since all share Ack; a read's client already got the refusal and its handler output, so its mail now comes back at the next read instead of staying reserved.
Ack's doc comment states the rule.

Test seam: the struct `journalWriter` gains an unexported field `writeFault func() error`, nil in production; `journalWriter.writeLine` calls it first when set and returns its error without writing.
`Store.FailJournalWrites(after int)`, declared in `internal/creelengine/export_test.go` (package `creelengine`, tier0 like that file), installs under the store's mutex a fault that lets `after` more writes pass and fails every later one with an error naming the injected fault, and removes the fault for a negative `after`.

Test (tier0): `TestStore_ClaimAtBoundary` gains the row `AJournalFailureAtTheAckReleasesEveryIdItLeftUnjournaled`: three short posts to beta, a tool-boundary claim of all three, `FailJournalWrites(1)`, Ack returns an error; after `FailJournalWrites(-1)` the log to beta holds one `read` event, the one of #1, and a new claim hands out #2 and #3, which would stay reserved and come back empty without the change.

`internal/creelengine/doc.go`'s claim paragraph states that a failed journal write at the acknowledgement releases the ids after it, so a later claim hands them out once more.

**ImpactSummary:** One method's failure branch and one unexported test seam in the journal writer; callers (handOut for read and claim) keep their signatures, and one test row pins it.
