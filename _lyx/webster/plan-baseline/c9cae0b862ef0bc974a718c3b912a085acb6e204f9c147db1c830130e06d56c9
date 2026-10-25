# Card 26 — batten-pass-exit-codes

**Edit:**
- `internal/battencli#ownedElsewhereRefusal`
- `internal/battencli#battenCLI.refuseResumeUnreachable`
- `internal/battencli#battenCLI.refuseCovered`
- `internal/battencli#battenCLI.driveRun`
- `internal/battencli#battenCLI.resolvePersistentPreRun`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `internal/battencli#TestArmAt_RunOwnedElsewhereWritesNoSeed`
- `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug`

**Create:**
- `plan:internal/battencli#OwnedElsewhereError` -> `type OwnedElsewhereError struct`
- `plan:internal/battencli#OwnedElsewhereError.Error` -> `func (e *OwnedElsewhereError) Error() string`
- `plan:internal/battencli#coveredRefusal` -> `type coveredRefusal struct`
- `plan:internal/battencli#passExitCode` -> `func passExitCode(err error) int`
- `plan:internal/battencli#TestPassExitCode` -> `func TestPassExitCode(t *testing.T)`

**Uses:**
- `internal/dobbyengine#PassExitOwnedElsewhere`
- `internal/dobbyengine#PassExitCovered`

**Intent:**
Give each refusal of a pass that dobby reads by code an exit code of its own (D4).
`OwnedElsewhereError` carries the slug and the owner and prints `ownedElsewhereRefusal`'s text, unchanged; `ownedElsewhereRefusal` returns it, so every refusal that reads another machine as the owner returns the one type: `resumeRefusal`'s `EntryOwnedElsewhere` arm, `resumeOwnerClaim`'s lost claim, `claimBoardEntry`'s lost claim and `refuseStartOwnedElsewhere`; `refuseResumeUnreachable`'s refusal on a local Bolt showing another owner returns it too, with its own "re-run once origin answers" text kept as the error's text.
`refuseCovered` returns a `coveredRefusal` wrapping `pauserecord.CoveredError`'s error, its text unchanged.
`driveRun` prints the pre-run's refusal envelope as today and sets the exit through `passExitCode`: `dobbyengine.PassExitOwnedElsewhere` for an error carrying `OwnedElsewhereError`, `dobbyengine.PassExitCovered` for a `coveredRefusal`, and 1 for any other; the codes are set only for a refusal before the step loop, never from a row's error, since a row's error ends in the driver's envelope and its own exit.
A first pass with no status file meets `refuseStartOwnedElsewhere` inside `armAt`, which `resolvePersistentPreRun` calls before `driveRun` exists, so `resolvePersistentPreRun` aborts with `clihelp.Abort(ctx, passExitCode(err))` for the run verb instead of 1, and that refusal exits with the owned code too (D4).
A claim the Worktree-Create row loses fails that row with the refusal's text, so the stop goes through the shed chain like any row failure, and a hand pass of the run then reaches `refuseStartOwnedElsewhere` at the found Worktree-Create status (`ownerPreflight`) and exits with the owned code.
Tests: `TestPassExitCode` (tier0, parallel) rows: each owned-elsewhere refusal source the pre-run reaches (`resumeRefusal`'s `EntryOwnedElsewhere` arm, `resumeOwnerClaim`'s lost claim, `refuseStartOwnedElsewhere` and `refuseResumeUnreachable`'s), wrapped by the pre-run, reads 77, while `claimBoardEntry`'s lost claim, which happens only inside the Worktree-Create row, is wrapped by no pre-run row; a covered refusal reads 78, a plain error reads 1; `TestRefuseStartOwnedElsewhere` asserts the returned error is an `*OwnedElsewhereError` naming the owner; `TestBattenRun_RefusesWhileAPauseCoversTheSlug` drives the run verb's `driveRun` through a cobra command run by `clihelp.Execute`, in place of its direct `battenPreRun` call, and asserts the returned exit code is 78 with the envelope's refusal naming `lyx resume`, and the same `driveRun` harness gains a row whose Worktree-Create row fails with an `OwnedElsewhereError`, asserting the exit is the driver envelope's own code, not 77, since `driveRun` applies `passExitCode` only to the pre-run's refusal; its window arm stays until card 28 removes it; `TestArmAt_RunOwnedElsewhereWritesNoSeed` asserts the run verb's recorded abort code is 77.

**ImpactSummary:** The owner refusals and the pause refusal change type, not text, and the run verb maps them to two exit codes, at the pre-run and at the arm's abort; every other exit stays 0 or 1.
