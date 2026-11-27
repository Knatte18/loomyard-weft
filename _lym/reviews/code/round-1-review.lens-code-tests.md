```yaml
lens: code-tests
outcome: reported
```

Task: run the code-tests lens over the diff, finding for each added or changed behaviour the test row that would fail if it broke.

**Per behaviour, the test that pins it**
1. A probe that fails in transit runs again and the Add succeeds once origin answers (`retryOriginProbeInTransit` loop, Add wiring): `TestAdd_RetriesOriginProbesThatFailInTransit/OriginAnswersAgain` pins it. Without the retry, Add returns the first refusal and the row fails at `Add() = …; want nil`.
2. Once the delays are spent, the last transport failure is returned and nothing is created, for both probes (warp and weft): `TestAdd_RetriesOriginProbesThatFailInTransit/OriginStaysDown` pins it with `wantSleeps` 6 and `requireNothingCreated`. Only this row shows both probes are retried. Any count other than three re-runs per probe fails it.
3. A failure that is not a transport failure returns at once with no re-run (`if !gitexec.IsTransportFailure(err) { return err }`, `add.go:95`): no test. If non-transport errors were retried too, every existing refusal test would still pass, only 7 s slower in real sleep, since those tests leave `BackoffSleep` nil.
4. Each re-run starts the probe from scratch and overwrites `warpTip`/`weftProbe` (`add.go:292-306`): pinned indirectly by row 1, where the pair is made from the second probe's answer.
5. `AddOptions.BackoffSleep` now also drives the probe backoff: pinned by both rows, which count sleeps through the hook.
6. `primeShuttle` with no task worktree skips the usage recorder and logs no warning (`drive.go:535-546`): no test. `TestPrimeShuttle_WiresTheReplyProbe` (`wire_seams_test.go:302`) runs with no task worktree but asserts only the reply probe. It would pass before and after the change.
7. `primeShuttle` with a present task worktree still wires `UsageRecorder`: no test. The refactor nests this path under `present`, and nothing would catch a wiring that never calls `SetSessionEnd`. `Runner` exposes no accessor for the session-end hook (`shuttleengine/sessionend.go:34`), so no test can read it today.
8. `primeShuttle` with a failing `taskWorktreePresent` or `taskWorktreeLocation` logs a warning and leaves the sessions unrecorded: no test.
9. The new sentences in `battenTacklerRecipe` and `battenArbiterRecipe`: static prose, exempt.

**Findings**

- **MEDIUM** · design (test economy: uncovered behaviour) · `internal/battencli/drive.go:535`
  The fix's second change, that an absent pair before Worktree-Create gives no warning and the present-pair path still wires `UsageRecorder`, has no test that fails if it breaks.
  Evidence: `TestPrimeShuttle_WiresTheReplyProbe` asserts only `HasReplyProbe` (`internal/battencli/wire_seams_test.go:327`). `Runner` has no getter for `sessionEnd` (`internal/shuttleengine/run.go:61`, `sessionend.go:34`).

- **LOW** · design (test economy: uncovered behaviour) · `internal/fabricengine/add.go:95`
  No row pins that a probe error which is not a transport failure is returned with no backoff. A retry-everything regression passes every test.
  Evidence: the retry test's only failure is `connection refused` (`add_remoteleftover_integration_test.go:136`). The existing refusal tests set no `BackoffSleep` that counts calls.

- **LOW** · consistency · `internal/fabricengine/add_remoteleftover_integration_test.go:141-150`
  The `BackoffSleep` hook calls `gitkit.Git(t, …)` from Add's probe goroutines. `gitkit.Git` calls `tb.Fatalf` when it fails (`internal/gitkit/gitkit.go:64-67`), and `FailNow` is not allowed off the test goroutine. A failed `set-url` there would leave the test in an undefined state instead of a clean failure.

- **NIT** · consistency · `internal/fabricengine/add_remoteleftover_integration_test.go:122`
  The `wantSleeps` value 0 in the `OriginAnswersAgain` row is never read. That row checks `sleeps.Load() == 0` instead, so the table field reads as if it asserted something.

Tool calls:
1. Grepped `shuttleengine` for a session-end accessor, `testkit`/`battencli` for a log-capture kit, and read `gitkit.Git`'s body. The context lacked whether `primeShuttle`'s recorder wiring can be observed, and whether `gitkit.Git` calls `Fatalf`.
