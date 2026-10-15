# Lens review — creel stage 1 plan

Every lens fork returned a report.
Findings repeated across lenses are merged and listed once, under every lens that raised them.

## Findings

- criteria-coverage, BLOCKING — `inject --interrupt` not refused under a pause or another sender's hold (D10): fixed as proposed; card 9 now runs the pause and hold refusals for both interrupt forms before anything is queued or sent, with rows for each form in `TestScheduler_Injects`.
- criteria-coverage MEDIUM + branch-coverage MEDIUM — the interrupt verb's privilege and orch refusals had no row, and which card owned them was unclear: fixed as proposed; card 7 owns the privilege and orch checks of `Interrupt`, card 9 the pause and hold refusals, and `TestStore_InjectAndHold` gains the non-privileged and orch interrupt rows.
- criteria-coverage, MEDIUM — no `orch-N` row: fixed as proposed; cards 7 and 9 add `orch-2` rows for inbox, self-inject, inject and interrupt refusals, and never-typed reminders.
- criteria-coverage MEDIUM + branch-coverage MEDIUM — no row for a reply by a caller that neither holds nor has read the message: fixed as proposed; a `TestStore_Mail` row.
- criteria-coverage MEDIUM + branch-coverage MEDIUM — the CLI exit codes for unconfirmed, expired and returned-undelivered were untested: fixed as proposed; `TestCreelCLI` rows over `creelfake` for each code.
- criteria-coverage, MEDIUM — no CLI row for `read` with nothing unread, and no row for `read --wait` waking: fixed as proposed; a `TestCreelCLI` row for the empty read, and a `TestDaemon_Protocol` row for `read --wait` returning once a post arrives.
- criteria-coverage MEDIUM + branch-coverage MEDIUM — `inbox` did not show an orch's reminders as withheld (D16): fixed as proposed; card 7's `Inbox` states the withheld reminders for an orch caller, with orch and `orch-2` rows and a log row.
- criteria-coverage, MEDIUM — D19's poll and waiting verbs were missing from the `PATTERN-daemon-wakeups` background, and the client's sub-second backoff would trip the scan: fixed as proposed; card 10 edits the background and paces the client's retries from one second on the told clock.
- criteria-coverage MEDIUM + row-reachability MEDIUM — card 14's test needed a fake delivery that `internal/creelcli` cannot import: fixed as proposed; card 7 creates the `internal/testkit/creelfake` kit, which card 9, card 10 and card 14 use.
- criteria-coverage, MEDIUM — no rows for D19's probe cadence or its Warn once per change of error: fixed as proposed; `TestScheduler_Reminders` reads the 1, 2, 4, 8, 8 second waits and the reset, `TestScheduler_Injects` counts the Warn lines through `logcapture`.
- criteria-coverage, LOW — no row for the 10-minute default TTL: fixed as proposed; a `TestStore_InjectAndHold` row.
- criteria-coverage LOW + branch-coverage MEDIUM — no rows for a non-privileged hold, release by a non-owner, or release with no hold: fixed as proposed; card 7 specifies the last two and adds a row for each of the three.
- criteria-coverage, LOW — a program mailbox known after its first read had no row: fixed as proposed; a `TestStore_Mail` row.
- criteria-coverage, LOW — a program claim of `EXTERNAL` was not refused (D4): fixed as proposed; card 8 refuses it on every platform, with a row.
- criteria-coverage, LOW — `EndedUnreadAt` named a time where D15 names the receiver: fixed as proposed; card 6's field is `EndedUnreadBy`, the receiver with its time, asserted in the fold and sweep rows.
- criteria-coverage, LOW — a malformed middle line fails every load with no way forward: fixed differently. The proposal to skip the line was not taken, because skipping it could drop a message silently, which D6's "the journal and the pool are one source" rules out. Instead the refusal names the file, the line and the way forward, to repair or remove that line by hand.
- criteria-coverage, LOW — a cancelled daemon stands in for a killed one: fixed differently. A real kill was not proposed, since an untagged test may not spawn and kill a process (`PATTERN-test-speed`). Instead `TestDaemon_Lifecycle` starts a daemon over a journal as a killed daemon leaves it: its last `aged` event a minute old and no stop-time record. "Restarts on the next verb" is covered by the client spawn rows.
- criteria-coverage LOW + branch-coverage LOW — `post --reply`'s argument count was ambiguous: fixed as proposed; card 14's `Use` is `post [<receiver>] <text>`, it takes two arguments without `--reply` and one with it, and a row refuses each wrong count.
- branch-coverage, MEDIUM — a record-less send into a busy pane had no row, nor a dead record-less strand: fixed as proposed; `TestSend_WaitsForIdleSession` gains record-less idle and busy rows, and `TestRun_InterruptAndSend_RefuseDeadOrUntrackedStrand` gains a record-less runner variant.
- branch-coverage MEDIUM + forced-edits LOW — it was undecided whether a damaged run.json falls back to the record-less path: fixed as proposed; card 5 falls back only on `ErrNoRunRecord`, the clean not-found answer, refuses on the unreadable-directories answer, and adds a row for it.
- branch-coverage, MEDIUM — waiting-verb reconnects running out, and returned-undelivered on reconnect, had no rows: fixed as proposed; two `TestDaemon_Lifecycle` rows.
- branch-coverage, MEDIUM — a worktree that does not resolve, a non-`ErrNoSession` status error and a pause-read error were unspecified and untested: fixed as proposed; card 13 fails `Strands` naming the worktree (not tendercli's skip, which D15 rules out), with a row for each of the three.
- branch-coverage, LOW — card 1 sent three-segment and invalid-shortname rows through `FormatProgram`: fixed as proposed; they go through `ParseProgram`, and a lower-case `FormatProgram` name joins.
- branch-coverage, LOW — no row for `ParentPID` of a pid that does not exist: fixed as proposed; the waited-on `sleep` child's pid.
- branch-coverage, LOW — no row for an empty journal or an unknown event name: fixed as proposed; card 6 specifies both (empty pool; unknown event ignored), with rows.
- branch-coverage, LOW — no rows for the hold owner's interrupt, a non-sentinel send error, or returned mail marked with its receiver: fixed as proposed; card 9 rows.
- branch-coverage, LOW — no rows for a malformed request or retention run from the poll: fixed as proposed; card 10 specifies the error response, and both get rows.
- branch-coverage, LOW — no row for a lyxtemp root that cannot be opened: fixed differently. A row that fakes `ProcessRoot` failing was not taken, since it would mean mutating `TMPDIR` under the test's run directory. Instead card 12's `CreelGeometry` is told the root path and opens nothing; card 14's open failure is a propagated raw I/O error passed through, which takes no row.
- branch-coverage, LOW — no rows for an absent `--hub-path`, a held lock or a `ResolveMode` error: fixed as proposed; `TestCreelCLI` rows.
- row-reachability, MEDIUM — the pending-send guard was unreachable with `readinessEngine`, which does not paste, and the assertion looked in the wrong directory: fixed as proposed; the rows run over `scriptedPasteEngine` and `clearableBoxEngine` and look under the run root and in the working directory.
- row-reachability, MEDIUM — the "no clear" assertion could not fail without `InputBoxClearer`: fixed as proposed; the row runs over `clearableBoxEngine`.
- row-reachability, LOW — the draft-hold follow-up needed a capture that is not idle: fixed as proposed; the row states it.
- row-reachability, MEDIUM — read rows ran as EXTERNAL, which is refused: fixed as proposed; cards 10 and 14 script a `creelfake` strand whose `PanePID` is `os.Getpid()`, and card 8's walk starts at the peer pid itself.
- row-reachability, MEDIUM — the adapter's error-mapping rows could not produce the shuttle sentinels through a real runner: fixed as proposed; card 13's `wire` seam returns `worktreeOps`, which the test stubs per call.
- row-reachability LOW + test-economy MEDIUM — the CLI and client rows waited out real time: fixed as proposed; the client paces and times out on a told clock, the CLI's clock is a package seam, and every row runs on `creelfake`'s clock.
- row-reachability LOW + branch-coverage LOW + test-economy LOW — `PanePID`'s dead-pane case was untested, and the assertion sat in a misnamed test: fixed as proposed; card 4 asserts in `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`, whose fixture holds the dead `%6`.
- row-reachability, LOW — `Replaced`'s Info log was unchecked: fixed as proposed; the change-seen-twice row asserts it through `logcapture`.
- forced-edits, MEDIUM — `newRoot`'s `Long` must name `creel` or `clitree_test.go` fails: fixed as proposed; card 14's Intent says so.
- forced-edits, LOW — the `lyx shuttle send` and `interrupt` help went stale: fixed as proposed; card 5 lists `internal/shuttlecli#shuttleCLI.sendCmd` and `#shuttleCLI.interruptCmd` and widens both texts.
- forced-edits, LOW — card 11 left open which `PATTERN.md` entries to cut, and whether a test pins them: fixed as proposed; card 11 names six entries with background files, in order, and states that no test pins entry wording (`grep` of the tests reading `PATTERN.md` shows only fixtures).
- test-economy, MEDIUM — `TestRunner_RecordlessStrand` duplicated existing tables: fixed as proposed; its rows are rows of `TestRun_Interrupt_PlaysEscape`, `TestRun_InterruptAndSend_RefuseDeadOrUntrackedStrand`, `TestSend_WaitsForIdleSession` and `TestRun_Send_ConfirmsSubmission`, and the shuttleengine budget raise is dropped.
- test-economy, LOW — `SocketName` was pinned twice: fixed as proposed; it is pinned only by `TestCreelGeometry`.
- test-economy, LOW — `TestClient_StartsDaemonOnDemand` repeated `TestDaemon_Lifecycle`'s lock row: fixed as proposed; its rows fold into `TestDaemon_Lifecycle`, and card 10's budget raise drops to two.
- test-economy, LOW — the reply-to-waiting-EXTERNAL row appeared in two cards: fixed as proposed; it stays in card 10's protocol test, and card 7 keeps the no-waiter row.
