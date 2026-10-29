# Card 3 — ack-failure-releases

**Edit:**
- `internal/creelengine#Reading`
- `internal/creelengine#Reading.Ack`
- `internal/creelengine#Store.Read`
- `internal/creelengine#journalWriter`
- `internal/creelengine#journalWriter.writeLine`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine/doc.go`

**Create:**
- `plan:internal/creelengine#Store.FailJournalWrites` -> `func (s *Store) FailJournalWrites(after int)`

**Intent:**
Decision D3's ordering rule: a journal write that fails at an acknowledgement releases every id the acknowledgement left unjournaled, where today `Reading.Ack` returns at the failing id with that id and the ones after it still in `Store.reserved`, so no later claim or read hands them out until the daemon restarts.

`Reading.Ack`: when `journalReadLocked` fails for an id of a claim's reading, Ack deletes the reservation of that id and of every id after it, answers the holder's mail waiters as `Reading.Release` does when it released any, clears `r.ids` and returns the error.
The ids before the failing one stay journaled read.
The struct `Reading` gains the unexported `read bool`, set by `Store.Read`, and a read's reading keeps today's failure branch: Ack returns at the failing id with it and the ones after it still reserved, since D3 releases on a claim only and the delivered-once Constraint names no exception for the read verb, whose client already printed the mail.
Every reading `Store.Read` does not make is a claim's, so the ring's reading of card 4-ring-claims-mail takes the release with no further change.
Ack's doc comment states the rule.

Test seam: the struct `journalWriter` gains an unexported field `writeFault func() error`, nil in production; `journalWriter.writeLine` calls it first when set and returns its error without writing.
`Store.FailJournalWrites(after int)`, declared in `internal/creelengine/export_test.go` (package `creelengine`, tier0 like that file), installs under the store's mutex a fault that lets `after` more writes pass and fails every later one with an error naming the injected fault, and removes the fault for a negative `after`.

Test (tier0): `TestStore_ClaimAtBoundary` gains the row `AJournalFailureAtTheAckReleasesEveryIdItLeftUnjournaled`: three short posts to beta, a tool-boundary claim of all three, `FailJournalWrites(1)`, Ack returns an error; after `FailJournalWrites(-1)` the log to beta holds one `read` event, the one of #1, and a new claim hands out #2 and #3, which would stay reserved and come back empty without the change.
`TestStore_Mail` gains the row `AJournalFailureAtAReadsAckKeepsTheIdsItLeftUnjournaledReserved`: three short posts to beta, beta's `Read` of all three, `FailJournalWrites(1)`, Ack returns an error; after `FailJournalWrites(-1)` the log to beta holds one `read` event, the one of #1, and a new `Read` hands out nothing.

`internal/creelengine/doc.go`'s claim paragraph states that a failed journal write at a claim's acknowledgement releases the ids after it, so a later claim hands them out once more, and that a read's keeps them reserved.

**ImpactSummary:** One method's failure branch, narrowed to claims by one unexported field Read sets, and one unexported test seam in the journal writer; callers (handOut for read and claim) keep their signatures, and two test rows pin it.
