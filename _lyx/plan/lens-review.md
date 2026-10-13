# Lens review

All four forks returned a report.

- row-reachability, BLOCKING, card 7 "status file gone" WARN row asserted every `producer stuck` line WARN though the running arm reports INFO first: fixed; each row asserts only the line carrying its cause's reason, and the row scripts a running status then a not-found `ReadStatus`.
- row-reachability, MEDIUM, card 4 did not pin release-before-respawn: fixed; the respawn fake takes the hub lock when called and records it.
- row-reachability, MEDIUM, card 8 agent-activity row never reached `agentActivity` with `newInnerRunDeps`' dead driver: fixed; that row sets `DriverAlive` true and notice rows wire `Notify` and `PairDir`.
- row-reachability, LOW, card 8 parent-lock row needs a `ParentLock` deps value: fixed; the card names its fields.
- row-reachability, LOW, card 10 shared activity set across one send's callers: rejected; card 10 already points both `newSendContext` constructors at the runner's set, which is the sharing the row relies on.
- row-reachability, LOW, card 5 second unreadable session setup unspecified: fixed; a second `writeSessionTranscript` for `sid2`, replaced with a directory.
- test-economy, MEDIUM, card 8 re-drove the reviewer-note and driver-strand failures existing tests reach: fixed; `TestInnerRun_RunningArmReviewWaitNote` and `TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn` are extended instead.
- test-economy, LOW, card 7 WARN rows re-asserted reason files the producer tests pin: fixed; WARN rows assert the level only.
- test-economy, LOW, card 1 test checks a value, not behavior: rejected; `out` is captured at init, so swapping `os.Stderr` cannot observe it, and the default writer is the observable at this level.
- test-economy, LOW, card 15 Land row repeats `rowNotifier.send`'s Warn branch: rejected; no existing row asserts that Warn, and the criterion is about a landing's notice, which only the Land route pins.
- test-economy, LOW, card 11 probe-route row overlaps the hold dedupe test: rejected; the probe route's `Draft` to constant-reason mapping in `heldReason` is new behavior no test reaches.
- criteria-coverage, MEDIUM, most D11 guard sites had no row: fixed; card 8 has a row per guarded and demoted site, card 11 a netops row in the hub-pause test.
- criteria-coverage, MEDIUM, "under go test the retirement spawns nothing" untested: fixed; the respawn seam takes `suppress`, RunE passes `testing.Testing()`, and the row asserts it true.
- criteria-coverage, MEDIUM, foreign-box-text send route never reached `heldTyping`: fixed; card 11 adds `ErrForeignBoxText` and `ErrSentTextInDraft` send rows.
- criteria-coverage, MEDIUM, archive Warn, Info and unchanged teardown result unasserted: fixed; two `archivePairLogs` rows assert the Info and Warn lines, and the void `archive` call before an unconditional removal is stated with the call-sequence rows.
- criteria-coverage, LOW, a new `Since` logging again unpinned: fixed; a `sinces` column and row.
- criteria-coverage, LOW, `driver_busy` still naming the draft argued only: fixed; card 9 cites `internal/loomcli/resume_test.go` and `internal/battencli/wire_test.go`.
- criteria-coverage, LOW, card 6 adds no test: rejected; a dropped trace field no reader parses is no behavior a test can pin.
- criteria-coverage, LOW, loop keeps running on unchanged or unstattable binary pinned only at the predicate: rejected; the loop's continue path is unchanged code and `exeReplaced` decides every case.
- criteria-coverage, LOW, audit table missed lines in `run.go`, `hold.go` and the orch watcher: fixed; each now has a keep verdict.
- forced-edits, MEDIUM, `internal/orchengine/doc.go` describes the old disagreement rule: fixed; added to card 11.
- forced-edits, MEDIUM, `NewInnerRun`'s arm table says the halt Warns: fixed; added to card 7.
- forced-edits, LOW, `innerrun_halted_test.go`'s header comment says Warns: fixed; named in card 7's Intent.
- forced-edits, MEDIUM, `reedengine` name repair's operator-activity Warn missing from the audit: fixed; guarded in card 3 with a row in `TestRepairNames_SessionName`, and listed in the table.
