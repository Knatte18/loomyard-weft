# Lens review

Every lens returned a report; the reports are saved as `lens-review.lens-<lens>.md` beside this file.
Findings raised by several lenses are merged into one line.

- coverage 1, guard-reach 1 — BLOCKING — fixed differently: not `Path[0]`, which a sweep's return empties (`internal/creelengine/fold.go:272-281`); card 3 adds `DeliveredTo`, card 7 compares it, and a row forwards between two deliveries.
- forced-edits 1, forced-edits 7 — BLOCKING, LOW — fixed as proposed: card 1 edits `TestSetEnv`, drops the new test, sets `LOCALAPPDATA` on Windows, and extends `PATTERN-test-isolation.md` and the `setTempEnv` comment.
- forced-edits 2 — BLOCKING — fixed as proposed: cards 4, 5, 6, 7, 8 and 9 spell each earlier card's symbol as its `plan:` handle.
- row-validity 1 — BLOCKING — fixed differently: not an edit of the rows on card 8, since `newHubPair` parks the worker from card 7 with a `Hubs` geometry error, so the rows hold across card 8 and only `TestScheduler_Outbound` calls `unpark`.
- way-forward 1 — MEDIUM — fixed as proposed: card 4 adds `daemon.recordHub`, once a minute on a served local verb, with a `TestDaemon_Lifecycle` row; the row sits there and not in `TestRecord`, which no change reaches.
- way-forward 2, guard-reach Lookup — MEDIUM — fixed as proposed: cards 1 and 4 add `ErrUnresolved` and `ErrHubUnresolved`, card 7 treats any other error as transient, with a row.
- way-forward 3 — MEDIUM — fixed as proposed: card 7 fixes a clash reason that fits 200 bytes whole, card 1 opens the several refusal with its rule, and card 8 has a row.
- way-forward 4 — MEDIUM — fixed as proposed: a failed settle takes the transient wait, a failed drop journal returns an error, each with a row on a `newEnv` store.
- way-forward 5, coverage 25, guard-reach store fault, row-validity 2 — MEDIUM, LOW — fixed as proposed: the row drives `Store.Deliver` on a `newEnv` store with `FailJournalWrites`, reworded as a failed store and a retry.
- way-forward 6, coverage 6, guard-reach nil seam, row-validity 7 (nil seam) — LOW, MEDIUM — fixed differently: not a refusal naming the cause on the nil seam; card 4's `daemonHubs` always passes a seam that answers the cause, so the nil refusal stays the wiring guard its text says, and no pending message can sit behind one in production.
- way-forward 7 — LOW — fixed differently: the record's wording stays and card 1 adds removing the extra clone as the route, since neither `lym fabric shortname` (`internal/fabriccli/shortname.go:39-43`) nor a clone of a weft that records one changes it.
- way-forward 8 — LOW — fixed as proposed: card 8 adds the `start-capped` code and one Warn naming the daemon log.
- way-forward 9, coverage 18 — LOW, MEDIUM — fixed as proposed: card 5's refusal names `lym config creel`, and rows reach it and the corrupt record.
- coverage 2, row-validity 7 (confirm, listing, Mail), coverage 7 — MEDIUM — fixed as proposed: card 7 rows for a sending daemon down, a timed-out and a neither-word confirm, a failed listing, a request with no `Mail` and an empty body.
- coverage 3 — MEDIUM — fixed as proposed: card 7 row over a real pending id differing in each tuple field.
- coverage 4 — MEDIUM — fixed as proposed: card 7 row for a sending shortname added twice to `Hubs`.
- coverage 5 — MEDIUM — fixed as proposed: card 7 row retries a stored message after its receiver left the listing and expects `accepted`.
- coverage 8 — MEDIUM — fixed as proposed: card 7 row changes `SetTrusted` between two arrivals.
- coverage 9 — MEDIUM — fixed as proposed: card 8 row for both resets of the start count.
- coverage 10 — MEDIUM — fixed as proposed: `TestHubDirectory` gains a `Spawn` row; the reap itself is `spawnDaemon`'s, unchanged, whose body `internal/creelcli/spawn.go:54` keeps out of tests.
- coverage 11, 12, 13 — MEDIUM — fixed as proposed: card 8 rows for the retry schedule, the `timeout`, `hub-unreadable` and `connect-failed` codes, and the order.
- coverage 14, row-validity 8 (waits) — MEDIUM — fixed as proposed: card 6 rows over journal lines and `reopen` reach the `WaitReply` and `settledOutcomeLocked` branches.
- coverage 15 — MEDIUM — fixed as proposed: card 3 states the fields `applyCarried` rebuilds, with rows for a chaseable inbound message and an original beside its carried line.
- coverage 16, 17, 24 — MEDIUM, LOW — fixed as proposed: card 9 rows for a reply through a chase, a wrong outbound sender and a forwarded holder.
- coverage 19, row-validity 4 — MEDIUM — fixed as proposed: card 8 seeds a pending line at `epoch` minus 40 minutes and asserts the return after 20.
- coverage 20, forced-edits 4, guard-reach CLI row, row-validity 10 — MEDIUM, LOW — fixed as proposed: card 9 edits `rig.startDaemon` to pass `daemonHubs`, and card 5 creates `rig.addHub`, which shares the one `seam`.
- coverage 21, 22, 23 — LOW — fixed as proposed: rows on cards 8 and 1.
- guard-reach body hash — MEDIUM — fixed as proposed: card 7 computes the hash from `Body`, with a forged row.
- guard-reach step 6, row-validity 6 — MEDIUM — fixed as proposed: `listenHubFake` holds confirms, so card 7's overlap, clash and lock rows are deterministic.
- guard-reach newHubPair, row-validity 3, row-validity 5 — MEDIUM — fixed as proposed: `hubPair` starts nothing, keeps the running handles, and `listenHubFake` scripts a receiver; the lost-`accepted` row states its mechanism, an inbound record beside a pending line past the hour.
- guard-reach sweep timers, row-validity 8 (sweep) — MEDIUM — fixed differently: not an edit of `isSweepWait`, which existing tests share; both rows use one `Advance` and assert the log.
- guard-reach no after overlap — LOW — fixed as proposed: card 7 checks the held messages on no, recorded in `## Shared Decisions`.
- forced-edits 3 — MEDIUM — fixed as proposed: card 9 edits `ComputeReplies`, with `LogEntry.Answers` from card 3 and a `TestComputeReplies` row.
- forced-edits 5 — LOW — fixed as proposed: card 3 states the `logEntryFor` branches, with rows.
- forced-edits 6 — LOW — fixed differently: cards 3, 6 and 7 move `doc.go` under `Edit`, the three the finding's lines show falsified; cards 4 and 8 stay under `Extends`, since no cited line is one they change.
- forced-edits 8 — LOW — fixed as proposed: cards 5, 6, 9 and 10 update the group `Long`, `postCmd`, `forwardCmd` and the orch role's line.
- row-validity 9 — LOW — fixed as proposed: card 4 states the fake's zero values.
- row-validity 11 — LOW — fixed as proposed: `TestHubDirectory` keeps only what no other test reaches.
- row-validity 12 — LOW — fixed as proposed: card 7 folds the two clash rows.
- row-validity 13 — LOW — fixed as proposed: card 1 records from many goroutines.
