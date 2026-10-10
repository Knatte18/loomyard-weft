# Card 22 — one-shell-wait-rule-and-ended-shells

**Create:**
- `plan:internal/shuttleengine#EndedShell` -> `type EndedShell struct`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Config`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#Run.expiredTurnEnd`
- `internal/shuttleengine#Run.recordWaiting`
- `internal/shuttleengine#Run.logPayloadShellPastBound`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#ShellWaitBound`
- `internal/shuttleengine#Run.shellWaitActive`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#heldTurnEnd`
- `internal/shuttleengine#holdNotice`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/waitmark_test.go`
- `internal/shuttleengine/waitshadow_test.go`
- `internal/shuttleengine/hold_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/shuttleengine/doc.go`
- `internal/websterengine#Run`
- `internal/websterengine#finishMasterDone`
- `internal/websterengine#AppendBackgroundShells` -> `func AppendBackgroundShells(websterDir string, shells []shuttleengine.EndedShell) error`
- `internal/websterengine#expiredShellWarning` -> `func expiredShellWarning(shell shuttleengine.EndedShell, outcome backgroundShellOutcome) string`
- `internal/websterengine#writeBackgroundShellFrictionNote` -> `func writeBackgroundShellFrictionNote(frictionDir string, shells []shuttleengine.EndedShell, outcome backgroundShellOutcome) error`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/summary_test.go`
- `internal/websterengine/triage_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/verbs_test.go`

**Delete:**
- `internal/shuttleengine#ShellWaitExpires`
- `internal/shuttleengine#noticeMaxTasks`

**Intent:** Make shuttle's wait apply one rule to a transcript-reported and a payload-reported shell, keyed on the output files, and record every shell a run ends over (D9).
An ungated run keeps today's rule: a turn end with every output file present counts at once, whatever is outstanding.
A turn end with an output file missing waits on every outstanding task, a shell of either signal included, until the run's deadline and the liveness check; `expiredTurnEnd` no longer expires a transcript-reported shell after the bound.
A gated run's waiting turn end with every output file present keeps today's deferral, with one exception: it counts at once when every outstanding task is an unawaited shell of either signal, the run was started fresh rather than attached or resumed, and no gated arrival of this run has reached the gate yet.
`Run` carries both facts as fields: the started-fresh fact, which `Runner.start` sets on the `Run` it builds only when the spec's `ResumeSessionID` is empty, since a resumed session's output files can predate the run, and which `Runner.reconstruct`, the one builder of the `Run` behind `Attach`, `AttachGated` and `AttachIfLive`, leaves unset without an edit; and a gated-arrival mark, which `Wait` sets at every gated Done arrival before it consults `boundaryIdle`, and which nothing clears.
`expiredTurnEnd` reads those two fields and never `gateFails`, `gateSent` or `gatePending`, since a failing arrival whose re-prompt send failed busy or unlanded (`unsentReprompt`), an arrival `boundaryIdle` deferred, a pending `PassOnCap` or `MayHold` answer and a pass that reset a failure count all leave those counts at zero while the files on disk may be that arrival's.
`Run` drops `expiredShells` and `expiredLabels`, the state of a shell already waited out, since nothing is waited out any more.
`ShellWaitBound` stays the threshold, for a shell of either signal, after which the shell is logged once and shown in the wait marker; it expires nothing, and `Config`'s field comment and the `background_shell_wait_min` comment in `template.yaml` say so.
`logPayloadShellPastBound`, whose comment widens to both signals, logs at Warn in place of today's Info, once per shell id, for every outstanding shell past the bound.
`shellWaitActive`, which read the expiry state, reports the shell wait on exactly while every outstanding task of the recorded waiting turn end is an unawaited shell of either signal and at least one of them has been outstanding for the bound by its first-seen stamp, so the wait marker shows the shell wait from the bound on, display only (PATTERN-wait-mark-display-only).
`expiredTurnEnd` now either counts a turn end at once, as a Done, or keeps it waiting, and never returns a held turn end carrying tasks, so `heldTurnEnd` drops its `tasks` field, `Wait`'s held-turn-end log drops its outstanding count, and `holdNotice` drops its outstanding-task listing together with the `noticeMaxTasks` cap it used, every held turn end it renders being a plain Stop or a live ask with none.
A fork, an awaited-prefix shell or an empty outstanding list holds a gated waiting turn end whatever the files, so `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` stands; every other gated waiting turn end waits as a missing-file turn end does, and only a later turn end with nothing outstanding is the arrival.
`Result.ExpiredShells` becomes `Result.EndedShells []EndedShell`, each entry holding the shell's label, id, signal and how long it was outstanding, recorded for each shell outstanding when the run ends, whatever ends it: an ungated or gated at-once finish, a later arrival, the run deadline or the liveness check.
`expiredTurnEnd` clears the waiting list when it counts a turn end at once, so a later tick never counts the same turn end twice, and keeps that list on `Run` as the counted turn end's shells first; `pollEventsTick` drops them when it reads a newer turn end, and `finalize` fills `EndedShells` from the waiting list and the first-seen stamps, or from the counted turn end's shells when no waiting turn end is recorded.
`finalize` logs the record at Info where it is true: beside a successful `RemoveStrand` on its cleanup branch, that the strand removal ended those shells; on every other end, a non-Done outcome, a `KeepPane` run or a failed removal, that the run ended with those shells outstanding and the strand is left to its caller.
`ShellWaitExpires` is deleted; card 21 removed its last caller.
Bound: lyx kills no shell, since the provider reports no pid; a shell detached from the pane's process tree survives the removal, and the record says the removal ended the shells it could.
A stale shell can now hold a run open until its deadline but never ends a turn early.
`websterengine` consumes the record: `Run`'s note closure and `finishMasterDone` read `EndedShells`, `AppendBackgroundShells` renders label, signal and duration per shell under a heading that no longer names the expiry, `expiredShellWarning` words one ended shell with the run's outcome and drops the bound, and `writeBackgroundShellFrictionNote` lists the same entries.
`shuttleengine`'s `doc.go` rewrites its background-shell passage and `websterengine`'s its `Background shells in Master's wait` section.
`TestPollEventsTick_ShellExpiry` and `TestWait_GatedShellExpiryEvaluatesGate` are reversed into the acceptance rows: a fresh gated run with output files present and a transcript-reported shell outstanding, before any gated arrival, finishes at once with the shell in the record; after a gate evaluation, a pending answer included, after a failing arrival whose re-prompt send failed busy, on an attached run, and on a run started with a `ResumeSessionID`, it waits on the shell; an ungated run with files present finishes at once whatever is outstanding; with an output file missing both signals wait past the bound, log once at Warn and end `OutcomeTimeout` at the deadline; an awaited-prefix shell and a fork keep waiting; and a gated at-once finish, an ungated at-once finish, a deadline end and a liveness end each record the shells outstanding in `EndedShells`, with the removal line captured on the cleaned Done ends and the outstanding line on the deadline and liveness ends, in shuttleengine's own rows.
`waitmark_test.go`'s shell-wait rows assert the marker off for a waiting turn end of transcript-reported shells before the bound and on past it, where the expiry used to turn it off, and off for one holding a fork or an awaited shell; `hold_test.go`'s notice rows drop the outstanding-task listing; `waitshadow_test.go`'s row named for transcript-reported shells that expired is renamed and reworded to a held turn end beside busy background work, since no shell expires.
`completionsignal_enforcement_test.go`'s audited function set changes with the rewrite, so PATTERN-completion-signal holds (the at-once branch and the deadline and liveness ends consult `allOutputFilesExist` before any negative verdict).
`TestRun_ExpiredShells` in `runlevel_test.go`, `TestAppendSummarySections`, `TestBackgroundShellNoteAndWarning` and `TestRunCmd_DiedMasterNotesExpiredShellOutcome` assert the record's rendering with signal and duration.

**ImpactSummary:** Shuttle's wait loop, run builders, hold notice and result change shape, and websterengine is the one consumer outside shuttleengine, its re-signed helpers and their callers on this card, plus one webstercli test; claudeengine's event parsing is untouched.
