# Card 21 — one-shell-wait-rule-and-ended-shells

**Create:**
- `plan:internal/shuttleengine#EndedShell` -> `type EndedShell struct`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Config`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#Run.expiredTurnEnd`
- `internal/shuttleengine#Run.recordWaiting`
- `internal/shuttleengine#Run.logPayloadShellPastBound`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#ShellWaitBound`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/wait_test.go`
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

**Intent:** Make shuttle's wait apply one rule to a transcript-reported and a payload-reported shell, keyed on the output files, and record every shell a run ends over (D9).
An ungated run keeps today's rule: a turn end with every output file present counts at once, whatever is outstanding.
A turn end with an output file missing waits on every outstanding task, a shell of either signal included, until the run's deadline and the liveness check; `expiredTurnEnd` no longer expires a transcript-reported shell after the bound.
A gated run's waiting turn end with every output file present keeps today's deferral, with one exception: it counts at once when every outstanding task is an unawaited shell of either signal, the run was started fresh rather than attached or resumed, and the gate has evaluated no arrival of this run yet, a pending `PassOnCap` or `MayHold` answer included; `Run` carries the started-fresh fact and `expiredTurnEnd` reads it with `gateFails`, `gateSent` and `gatePending`.
A fork, an awaited-prefix shell or an empty outstanding list holds a gated waiting turn end whatever the files, so `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` stands; every other gated waiting turn end waits as a missing-file turn end does, and only a later turn end with nothing outstanding is the arrival.
`ShellWaitBound` stays the threshold after which an outstanding shell of either signal is logged once at Warn through `logPayloadShellPastBound`, renamed in its comment to cover both signals, and shown in the wait marker; it expires nothing, and `Config`'s field comment says so.
`Result.ExpiredShells` becomes `Result.EndedShells []EndedShell`, each entry holding the shell's label, id, signal and how long it was outstanding, recorded for each shell outstanding when the run ends, whatever ends it: an ungated or gated at-once finish, a later arrival, the run deadline or the liveness check; `finalize` fills it from the waiting list and the first-seen stamps and logs at Info that the strand removal ended those shells.
`ShellWaitExpires` is deleted; card 20 removed its last caller.
Bound: lyx kills no shell, since the provider reports no pid; a shell detached from the pane's process tree survives the removal, and the record says the removal ended the shells it could.
A stale shell can now hold a run open until its deadline but never ends a turn early.
`websterengine` consumes the record: `Run`'s note closure and `finishMasterDone` read `EndedShells`, `AppendBackgroundShells` renders label, signal and duration per shell under a heading that no longer names the expiry, `expiredShellWarning` words one ended shell with the run's outcome and drops the bound, and `writeBackgroundShellFrictionNote` lists the same entries.
`shuttleengine`'s `doc.go` rewrites its background-shell passage and `websterengine`'s its `Background shells in Master's wait` section.
`TestPollEventsTick_ShellExpiry` and `TestWait_GatedShellExpiryEvaluatesGate` are reversed into the acceptance rows: a fresh gated run with output files present and a transcript-reported shell outstanding, before any gate evaluation, finishes at once with the shell in the record; after a gate evaluation, a pending answer included, and on an attached run, it waits on the shell; an ungated run with files present finishes at once whatever is outstanding; with an output file missing both signals wait past the bound, log once and end `OutcomeTimeout` at the deadline; an awaited-prefix shell and a fork keep waiting; and an ungated at-once finish, a deadline end and a liveness end each record the shells outstanding in `EndedShells`.
`completionsignal_enforcement_test.go`'s audited function set changes with the rewrite, so PATTERN-completion-signal holds (the at-once branch and the deadline and liveness ends consult `allOutputFilesExist` before any negative verdict).
`TestRun_ExpiredShells` in `runlevel_test.go`, `TestAppendSummarySections`, `TestBackgroundShellNoteAndWarning` and `TestRunCmd_DiedMasterNotesExpiredShellOutcome` assert the record's rendering with signal and duration and the end-of-run log.

**ImpactSummary:** Shuttle's wait loop and result change shape, and websterengine is the one consumer outside shuttleengine (three helpers re-signed, two callers), plus one webstercli test; claudeengine's event parsing is untouched.
