# Card 5 — shuttle-recordless-strand

**Edit:**
- `internal/shuttleengine#Runner.Send`
- `internal/shuttleengine#Runner.Interrupt`
- `internal/shuttleengine#findRunByStrand`
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#sessionReadiness`
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#pasteVerified`
- `internal/shuttleengine#TestRun_Interrupt_PlaysEscape`
- `internal/shuttleengine#TestRun_InterruptAndSend_RefuseDeadOrUntrackedStrand`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/shuttleengine#TestRun_Send_ConfirmsSubmission`
- `internal/shuttlecli#shuttleCLI.sendCmd`
- `internal/shuttlecli#shuttleCLI.interruptCmd`
- `//pattern/PATTERN-pane-text-injection.md`

**Create:**
- `internal/shuttleengine#ErrNoRunRecord`
- `internal/shuttleengine#Runner.recordlessSendContext`

**Intent:** Make shuttle's verified send and guarded interrupt work on a live reed strand that has no shuttle run record (D17), the path creel delivers through to a hand-added strand.
`findRunByStrand` wraps `ErrNoRunRecord` in its clean not-found answers (no run root, or every run.json read and none names the guid); its answer when some run directories could not be read does not wrap it, since that guid may name a live recorded run whose run.json is damaged.
`Runner.Send`: when `FindRun` fails wrapping `ErrNoRunRecord`, it no longer refuses; it builds `recordlessSendContext(guid)` and sends through `sendVerified`.
`Runner.Interrupt`: on the same answer it goes straight to `guardedInterrupt`, which keeps both pane guards (the ready-agent-pane check and the running-turn capture).
Any other `FindRun` error, the unreadable-run-directories answer included, still refuses as today, so a recorded run with a damaged record is never driven pane-only.
On the record-less path the strand must be live in reed: an untracked or dead guid keeps failing, and the error keeps the words "not a shuttle strand" beside reed's liveness reason (for example `shuttle: %q is not a shuttle strand and not a live reed strand: <liveness error>`), so `internal/webstercli#TestWire_StandaloneMode`'s assertion on an unknown guid stays true unchanged.
`sendContext` gains a `recordless bool` field; `recordlessSendContext` sets it, leaves the run state, run directory and events path empty, sets `keepUnlanded` and takes the runner's per-strand turn-start hold.
`sessionReadiness` answers unknown with cause `no-run-record` for a recordless context, so the idle wait takes the pane-only reading (no hook state, no events file), as `Runner.SessionIdle` already does for such a strand.
`sendVerified` skips `retryLanded` and `pasteVerified` writes no pending-send record for a recordless context, since there is no run directory; landing is confirmed by the pane alone, and an unlanded text is never cleared, so it stays in the box and a later send reads a non-empty box and holds with `ErrSessionBusy`.
The send is driven with the runner's own engine, the default one.
Bound: shuttle may now type into any live strand reed tracks in the runner's worktree, behind the same pre-write guard; a strand whose pane is not an agent reads not ready and gets nothing, and a text left in the box holds later sends until someone clears it.
Tests, as rows of the existing tables, each over a fixture built without `withStrand`, so no run record names the strand:
- `TestRun_Interrupt_PlaysEscape` gains a `recordless` field and two runner rows: a ready pane showing a running turn gets one Escape, and an idle pane gets no key and `ErrNoRunningTurn`.
- `TestRun_InterruptAndSend_RefuseDeadOrUntrackedStrand` gains a `viaRecordlessRunner` variant over its dead, cleared-binding and untracked rows: both `Runner.Interrupt` and `Runner.Send` fail naming "not a shuttle strand" and the row's liveness phrase, and no key or text reaches the pane; and a row whose run root holds one unreadable run.json beside a live strand fails both verbs naming the unreadable run directories, touching no pane.
- `TestSend_WaitsForIdleSession` gains a `recordless` field on its `viaRunner` path and rows for a record-less strand: a pane that turns idle takes the send inside the window, and a pane that stays not idle fails with `ErrSessionBusy`, nothing typed.
- `TestRun_Send_ConfirmsSubmission` gains record-less rows over `clearableBoxEngine` and `scriptedPasteEngine`: a paste that lands leaves no `pending-send.json` under the anchor's run root or in the test's working directory; a paste whose box still holds its text when the window closes fails with `ErrSubmissionNotLanded` and sends no clear sequence, though the engine can clear; and the next send, over a pane capture that is not idle and whose box holds that text, fails with a `SessionBusyError` whose `Draft` is true.
`internal/shuttlecli#shuttleCLI.sendCmd` and `internal/shuttlecli#shuttleCLI.interruptCmd` widen their `Short` and `Long` from "a shuttle run's pane" to a shuttle run's or live reed strand's pane.
`pattern/PATTERN-pane-text-injection.md` gains a bullet for the record-less send: a strand with no run record takes the pane-only readiness reading, keeps no pending-send record and never clears its unlanded text, so the next send holds on the non-empty box.

**ImpactSummary:** Send and Interrupt stop refusing record-less live strands; recorded runs, damaged records and the unknown-guid wording are unchanged, and two shuttle CLI help texts widen.
