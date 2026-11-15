```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `launchOrch`, a resume refused with `ErrNoTranscript` launches fresh, names the abandoned session and adds the fresh-launch warning with the carried count. Pinned by `TestStart_ResumesTheRecordedSession`, the two `noTranscriptErr` rows (`carried` 2 and 0, `wantAbandoned`, `wantWarning`).
2. `launchOrch`, any other failed resume fails the start and launches nothing fresh. Pinned by `TestStart_ResumeOverALiveHolder/AResumeThatFailsForAnotherCauseLaunchesNothingFresh` and by `TestStart_StarterErrorFailsAndSavesNoState/a failed resume`.
3. `resumeOrch` with a holder that is registered: the holder is ended and the same session resumed. Pinned by `TestStart_ResumeOverALiveHolder/ThisWorktreesOwnEarlierOrchIsEndedAndTheSameSessionResumed`.
4. `endOwnHolder` with a holder lym did not register: it refuses, names the pid and the way forward, and starts nothing fresh. Pinned by `.../AHolderLymDidNotRegisterRefusesNamingItsPIDAndLaunchesNothingFresh`.
5. `endOwnHolder` when `endOrch` fails: it refuses. Pinned by `.../AFailedEndOfTheOwnHolderRefusesAndLaunchesNothingFresh`.
6. `endOwnHolder` when the `orchRegistered` read fails: it refuses with a way forward. **None.**
7. `endOwnHolder` with `orchRegistered` nil: every pid reads as unregistered. None. This is a wiring default only, so I file no finding.
8. `resumeOrch` when the resume after ending the holder fails, for example still held: `resumeFailure` is returned and nothing launches fresh. **None.**
9. `registerOrchProcess` registers the holder pid. It warns when no holder is found and warns when registration fails. Pinned by `TestStart_RegistersTheLaunchedOrchProcess`, its three rows.
10. `registerOrchProcess` puts the `HolderErr` text into the warning. **None.** `fakeStarter` has no holder-error field.
11. `runnerSessionStarter.StartSession` maps `Run.SessionHolder` to `HolderPID` or `HolderErr`, and `len(CarriedInstructions)` to `Carried`. **None.** Every orchcli test replaces it with `fakeStarter`.
12. `shuttleengine.Run.SessionHolder` finds no holder for an engine without `SessionHolderFinder` or a run with an empty session id, and delegates otherwise. Same gap for `Run.CarriedInstructions`. **None.**
13. `checkResumable` wraps `ErrNoTranscript` and returns `*SessionHeldError`, for both proven and unproven holders. Pinned by `TestCheckResumable`, the `wantNoTranscript` and `wantHolder` rows. The `liveHolder` refactor is pinned by the existing reused-pid, dead-pid and mismatch rows.
14. `sessionHolder` finds a live matching holder, an unproven holder, a reused pid, a dead pid and another session's entry, and returns an error for an unreadable registry. Pinned by `TestSessionHolder`. Skipping unreadable or undecodable entries has no row (minor).
15. `RegisterOrch` and `processRecord` record the pid, role, start time, own group and no argv; `IsRegistered` checks role, pid and owner. Pinned by `TestRegisterOrch_IsFoundByItsOwnerRoleAndPID`.
16. `SelfRecord` still records `os.Args`. Pinned by the existing `record_test.go:43`.
17. `EndSubject` ends the runs first, then the subject's own records minus the dobby, then the dobby. It spares the adopt worker, the board sync and other owners, and goes on past a survivor with the errors joined. Pinned by `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`.
18. `endSubjectProcesses` (`internal/orchcli/watch.go`) selects the registered owners whose `AdoptSubject` is the beam, or `RootSubject` for the prime. It skips the subject's own owner, and it leaves alone and logs an owner whose subject cannot be read. **None.**
19. `ownerTick` calls End before Stop, a failed End stops nothing and returns an error, and an adopt in flight keeps everything. Pinned by `TestWatcher_EndsTheProcessesAndStopsTheOrchOfABeamOwnedElsewhere`.
20. `lym fabric anchor` spares `RoleOrch` when it ends dobbies. Pinned by `TestChangeAnchor`, both updated `end-dobbies` expectations.
21. `TestStart_StarterErrorFailsAndSavesNoState`, which the redundancy report lists as contained. It asserts no watcher spawn and the seeded state unchanged after a failed launch. Neither covering test checks those: `TestStart_ResumeOverALiveHolder` returns early when `wantCode != 0`, and `TestStart_ResumesTheRecordedSession` has no failing rows. Its "fresh launch" failure row is covered by neither.

**Findings**

1. MEDIUM, design, `internal/orchcli/watch.go:115-157` (`endSubjectProcesses`). Which runs the watcher ends for a subject owned elsewhere has no test. Nothing checks beam versus `RootSubject`, skipping the subject's own owner, or leaving an owner with an unreadable subject alone. D9's "ends every process … run strands" depends on this selection. Evidence: a grep of `*_test.go` finds no `endSubjectProcesses`. `TestEndSubject_…` (`internal/runproc/end_test.go:511`) is handed the run list directly.
2. LOW, design, `internal/orchcli/start.go`, `endOwnHolder`'s `orchRegistered` error branch. The refusal when lym cannot read whether the holder is registered has no row. The fake in `TestStart_ResumeOverALiveHolder` always returns a nil error (`start_test.go`, the `h.cli.orchRegistered` closure).
3. LOW, design, `internal/orchcli/start.go`, the second `StartSession` in `resumeOrch`. No row makes the resume after a successful end fail, which is what proves a failure there is not followed by a fresh launch. Evidence: every `resumeErrs` in `TestStart_ResumeOverALiveHolder` either ends after `held` or is `{held, nil}`.
4. LOW, design, `internal/orchcli/start.go`, `runnerSessionStarter.StartSession` and `registerOrchProcess`'s `HolderErr` branch. The holder read and the carried count that feed registration and the fresh-launch note are untested in production. A `HolderErr` cause never reaches the warning under test. Evidence: the grep finds no `runnerSessionStarter` in any test, and `fakeStarter` (`start_test.go`) has no holder-error field.
5. LOW, design, `internal/shuttleengine/run.go:985-998`. `Run.SessionHolder` (no finder, or an empty session id, gives none) and `Run.CarriedInstructions` have no test. Evidence: the grep finds no `.SessionHolder()` or `CarriedInstructions()` in any `_test.go`.
6. LOW, consistency, `internal/orchcli/start_test.go:1339`, `TestStart_StarterErrorFailsAndSavesNoState`. The redundancy report lists it as contained, but it pins what its covering tests never check: no watcher spawn and unchanged state after a failed launch, plus a failing fresh launch. Mark it `//testtiming:keep pins that a failed launch spawns no watcher and saves no state` directly above its `func Test…` line.
7. NIT, consistency, `internal/orchcli/start_test.go:1351-1352`. The "a failed resume" row fails with the plain error text "no transcript for that session", which does not wrap `ErrNoTranscript`. Read with D9 in mind, it suggests a missing transcript fails the start, when that case launches fresh. Use a cause such as "pane never came up".

**Tool calls**

1. Grep of `*_test.go` for `endSubjectProcesses`, `runnerSessionStarter`, `Run.SessionHolder`, `CarriedInstructions` and the new orchcli seams, plus `newStartHarness`'s defaults: my context did not show whether any test reaches those symbols.
2. Grep of runproc tests for `Argv`/`SelfRecord`, and of orchcli tests for `popError`: to confirm the `SelfRecord` refactor keeps an argv assertion and that the fake's helper exists.
