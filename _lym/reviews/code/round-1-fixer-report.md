```yaml
round: 1
review: _lym/reviews/code/round-1-review.md
fixed: [F1, F2, F3, F4, F5, F6, F8, F9, F10]
decision_class_left_to_parent: [F7]
```

## Changes

Commits `5ba881a1`, `00e4a532`, `38d29f15`, `a025f4c0`.
One commit per finding was not possible: `internal/gatecli/tierrun.go`, `cli.go` and `doc.go` carry hunks of several findings, and hunk staging is interactive.
So the commits group findings by file.

- **F1** (fixed as proposed, then refined by the self-check).
  `runMemoryCapped` reads `scopeHadOOMKill` before `killScope` and returns it as `treeKilled`; `killed` is unchanged, so `RunMemoryCapped` keeps its contract.
  Both doc comments say `treeKilled` is the kernel's OOM kill in the scope.
  `TestRunMemoryCappedVia_ASigkillFromOutsideIsNoKillAtTheCap` fails with the old `treeKilled = killed || oom`.
  The `stops.processCap == ""` guard the review proposed in `runInvocations` is not kept: see Fixed differently.
- **F2** (fixed with an own variant of the proposal's plumbing).
  `testcover.Toolchain.GuardKilled func(pid int) bool` and `RunResult.GuardKilled`; `RunAlone` returns a failed run the guard killed as `GuardKilled`, never `MemoryCapped`.
  `redundancy.go` lists such a run as not judged (`reasonGuardKilled`).
  `redundantmutants.go` records no kill for it, skips its rerun, and marks its candidate, and each candidate it covers, `GuardKilled`, never `FailsUnmutated`.
  `testmappackage.go` records it as failed with a logged reason.
  `testcover.MutantCandidate.GuardKilled` and `ReasonGuardKilled` carry it into `JudgeMutants`.
  Tests: two `TestRunAlone` rows, a `TestJudgeMutants` row and `TestRunMutantsGuardKilledRunIsNoKill`, which re-executes the gatecli test binary as the stand-in coverage binary.
- **F3** (fixed as proposed).
  `runTest` wraps the redundancy report step in `startMemoryGuard`, with the already-decided `exe.request.memoryGuarding`, and hands the step `guardKilled`.
  `doc.go`, `template.yaml`, `docs/overview.md` and `pattern/PATTERN-daemon-wakeups.md` name the step.
- **F4** (fixed as proposed).
  The go-json counter tracks tests that sent `run` and no `pass`, `fail` or `skip`, and `Unfinished()` returns them per package.
  `runTier` adds one cause line, in the log and the causes, naming those tests when a memory guard or `memory_mb` kill stopped the run.
  The premise was checked against a real `go test -json` run with the test binary SIGKILLed: the running test keeps its `run` event and gets no result event.
  Tests: a counter row and the `TestRunTierMemoryGuard` cause assertion.
- **F5** (fixed with an own variant).
  `proc.ErrMemoryCap` is wrapped by every error that means the cap could not be set or the scope never ran the command, on Linux and Windows.
  `runInvocations` refuses with the tier, its `memory_mb`, the error and the way forward when `errors.Is(runErr, proc.ErrMemoryCap)`, before the shell-start refusal.
  Tests: `TestRunTierMemoryCapThatCannotBeSetRefusesWithItsWayForward` (tier1, a host that caps a scope, run with no bus in its environment), `TestRunTierStartFailureKeepsTheShellRefusal` and the `proc` assertion that a starter that fails at once returns `ErrMemoryCap`.
- **F6** (fixed as proposed).
  `testRequest.memoryGuardFor` seam; `TestRunTierMemoryGuard` pins exit 1 on a zero exit, the line in the log, stderr and causes, `memoryStopped`, no rerun and no guard when the request turns it off.
  `TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap` (tier1, skips without a capping user manager) pins the `tierMemoryCapLine` cause.
- **F8** (fixed with an own variant).
  `guardOwnTests` keeps its signature and delegates to `startMemoryGuard(ctx, guard, root, applies)`, which the test calls with a scripted guard.
  `TestStartMemoryGuard` pins the no-op stop, the stop that ends a running guard, and the killed-pid answers.
- **F9** (fixed as proposed).
  The `.test`-only sentence is back at the end of the CPU watchdog section, and the Memory guard section says the guard watches `.test` processes while `memory_mb` caps every process of the command.
- **F10** (fixed as proposed).
  The `stderr` field comment names the memory guard's kills and a tier's memory cap line.

### Fixed differently

- **F1, `stops.processCap == ""`.**
  The review proposed setting `stops.memoryCap` only when the process cap did not stop the run.
  Not made: with `treeKilled` now the kernel's OOM count, a real OOM kill beside a process-cap kill would lose its line (fix-proposal lens, `tierrun.go`).
  `memoryCap` is set whenever the cap killed a process.
- **F2, F8.** The proposal's pid check became consume-on-read over the killed binary's pid only, noted before the SIGKILL.
  The self-check showed that a set of every killed pid, kept for the whole call, reads a later process that reuses a pid as guard-killed.
  `guardOwnTests` was not given the guard and decision as parameters: `startMemoryGuard` carries them, so the production signature and its callers stay.
- **F5.** The proposed `*exec.ExitError` test also refuses a tier that ran, when its counter output fails or `killScope` fails.
  The sentinel names only the cap failures.

### Gate repair

The gate found comment lines wrapped at a fixed column or at 120 characters or more.
They sit in `gatecli/doc.go`, `redundancy.go`, `tierrun_test.go`, `tierrun_memorycap_linux_test.go`, `proc/doc.go`, `memorycap_linux.go`, `testcover/mutantverdict.go` and `runner.go`.
Each is now broken at sentence boundaries, one sentence per line, with no change of meaning.

### Second gate repair

The verify run failed `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap`: one run in six read no OOM kill.
The scope is removed once its last process exits, and with it the `memory.events` count, so a read after the wait could find it gone.
`runMemoryCapped` now polls the count every 20 ms while the command runs and keeps the last read after the wait.
This removes the timing risk the first version of this report named for `treeKilled`.
The test passed ten runs in a row afterwards, and tier1 over `proc`, `testcover`, `gatecli`, `gateslot` and `cleanupcli` passes.
The `cleanupcli` failures in that verify run came from leftover read-only fixture directories in `/tmp/lym-1000` and not from the code; I removed the three it named.

## Disputed

None.

## Deferred

None.

Not changed, with the reason: the `Long` help text of `lym gate redundant` and `lym gate testmap` still reads "A test binary past memory_quit_mb is killed and named on stderr."
Any wording that adds the guard's effect on the test's verdict breaks the 2000-byte cap (`cmd/lym/clitree_test.go:413` reported 2009 and 2016 bytes).
The package doc and `docs/overview.md` carry the verdict.

## Rulings

- **F7** (parent ruling, `lym loom decision add`: tiers stay uncapped unless the hub sets `memory_mb`).
  The ruling is redundant: `gateslot.Tier.MemoryMB` defaults to 0, `internal/gateslot/template.yaml` declares no tier, and this round sets no hub value.
  Nothing changed.
  Flagged to the parent with `lym creel post --parent` (message 1542).

## Self-check

Four forks returned; reports are in `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, LOW: F2 wiring of `GuardKilled` in the mutation check untested. Fixed: `TestRunMutantsGuardKilledRunIsNoKill`.
- fix-code-correctness, LOW: F5 refusal can misfire after the command ran. Fixed: `proc.ErrMemoryCap`.
- fix-code-correctness, LOW: F2 pid reuse in `startMemoryGuard`. Fixed: consume-on-read, binary pid only.
- fix-code-correctness, MEDIUM: gateslot tests and `cmd/...` unrun after the `template.yaml` and `overview.md` edits. Fixed: ran `./internal/gateslot`, `./cmd/lym` and `./cmd/testtiming` at tier0, and the others at tier1.
- fix-consistency, LOW: `proc/doc.go` and `memorycap.go` say "any kill at the cap" where the scope is also ended after an outside SIGKILL. Fixed: both reworded.
- fix-consistency, LOW: the `Long` help of the two verbs is stale. Fixed differently: left as is, see Deferred.
- fix-proposal, LOW: F5 refusal fires on any non-`ExitError`. Same as the code-correctness finding, fixed by the sentinel.
- fix-proposal, LOW: F2 pid reuse. Same as above.
- fix-proposal, LOW: F2 guard kill of a covering test that is no candidate lists a candidate as refuted. Fixed: a candidate whose covering test the guard killed is marked `GuardKilled`.
- fix-proposal, LOW: F1 `stops.processCap == ""` drops a real kernel cap line. Fixed: guard removed.
- fix-affected-set: no findings.
  Its two out-of-scope notes: the `ProfileBlocks` guard kill surfaces as an error and ships no wrong verdict, left as is; `pattern/PATTERN-daemon-wakeups.md` listed the guard's sites, now updated.

## Verification

`lym gate test` tier0 over `./internal/proc ./internal/testcover ./internal/gatecli ./internal/gateslot ./cmd/lym ./cmd/testtiming` passed.
`lym gate test --tier 1` over `./internal/proc ./internal/testcover ./internal/gatecli ./internal/gateslot ./cmd/testtiming` passed.
`GOOS=windows go build ./internal/proc ./internal/gatecli` is clean.

Tier1 over `./cmd/lym` was not completed: `TestCSharpFixtures_CompileWithDotnet/internal/planglyph/testdata/glyphchain/csharp` ran to the 20-minute timeout on a `dotnet` compile, which this change does not touch.
That run also reported fixture-directory failures in `internal/gatecli` that did not repeat when the package ran without `cmd/lym`.

One timing note for the parent: `treeKilled` now rests on the scope's `oom_kill` count being readable once the command ends.
When the last process of the scope has already exited and systemd has removed it, the count reads false and a cap kill shows no cap line.
The same read decided every kill of a child before this round, so the defect is not new, but F1 now relies on it for a kill of the command itself too.
