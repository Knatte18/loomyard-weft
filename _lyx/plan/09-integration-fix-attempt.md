# Card 9 — integration-fix-attempt

**Edit:**
- `internal/websterengine/integrationfix.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/integration_test.go`
- `internal/webstercli/run.go`
- `internal/webstercli/cli.go`
- `internal/webstercli/wiring.go`

**Prosa:**
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine#RenderIntegrationFixPrompt`
- `internal/websterengine#IntegrationFixState`
- `plan:internal/websterengine#checkFixCommits`
- `plan:internal/websterengine#IntegrationFixRecord`
- `plan:internal/websterengine#AppendIntegrationFix`
- `internal/websterengine#triageIntegrationFailure`
- `internal/websterengine#RecordIntegrationFailure`
- `internal/websterengine#MasterHandle`

**Intent:**
An integration regression gets one automated fix attempt by a cold strand before the run escalates (#332b, decisions D5 and D6).

Seam and wiring:
- Add `FixStarter` to `integrationfix.go`: `StartFix(shuttleengine.Spec) (MasterHandle, error)`, two-phase like `MasterStarter` so the strand's GUID is persisted before Go blocks on it.
- `RunDeps` gains `FixStarter FixStarter` and `ParentBranch ParentBranchFunc` (nil in standalone mode, so no merge commit is accepted).
- webstercli adds a `runnerFixStarter` adapter over `Runner.Start` in `cli.go`, stores it in `setRunner` in `wiring.go`, and `runDeps` in `run.go` passes it and `c.parentBranch`.

`runIntegrationStage` keeps its three phases and adds the attempt after phase 2 (triage and bisect), when the verdict is a regression and Master's outcome is done.
A non-done Master outcome escalates as today with no attempt, since a fix cannot change that outcome.
The attempt lives in `integrationfix.go` as one function the stage calls; it never holds the state-mutation lease across the strand's wait or a verify run:
0. A nil `FixStarter` is a wiring-guard error, checked here before anything is recorded, so a wiring fault never spends the attempt.
1. Leased: reload state. A non-nil `IntegrationFix` means the attempt is spent: escalate without spawning, with result `spent`, naming the recorded `PreFixHead` and the commits from it to HEAD.
   Otherwise record `IntegrationFix` with `PreFixHead` (HEAD now) and `SpawnedAt`, and save.
   Capture the plan's fingerprint in memory.
2. Unleased: render the prompt with `RenderIntegrationFixPrompt` (the regressions with their tails, `plan.Verify`, the localized card's file as the hint when bisect named one, a fix report path under the reports directory, the plan directory) and start the strand at `RoleRecovery`'s model with `recovery_timeout_min` as its timeout and the report as its output file.
   A render error or a start error is a failed attempt (result `failed`, detail naming the error), recorded in phase 3 like any other.
3. Leased: record `StrandGUID` and save.
4. Unleased: wait on the handle; then remove the strand if live and its run dir, as recover-batch does.
   A timeout or dead outcome, a missing or malformed report, or `status: FAILED` is a failed attempt.
5. Unleased: a plan fingerprint that differs from step 1's, or an error from `checkFixCommits(worktree, PreFixHead, report head_sha, planDir, ParentBranch)`, is a failed attempt with result `refused`.
   The fingerprint compare catches an on-disk plan write no commit can show (card 7's bound).
6. Unleased: run the verify once at HEAD into a dedicated scratch log; on red, triage it with `triageIntegrationFailure` against the same start commits.
   No regression in the result is success: the post-fix triage decides, never the strand's report.

An attempt found in flight when the stage runs (a loaded `IntegrationFix` with an empty `Result`: the run died during the attempt) is checked after the integration report loads and before the `OK` early return, whatever the report's status.
It ends as result `failed` with a detail naming the interruption: the stage records `Fix` with the commits from `PreFixHead` to HEAD, appends `AppendIntegrationFix`'s section, sets `Result` in the same leased save, and returns a stuck reason carrying the way forward below.
So a resumed run never reaches done on commits `checkFixCommits` never saw.

`recordTriageResult` records the result in phase 3:
- Success: the report carries the pre-fix failures and triage plus `Fix` (result `fixed`, commits, cleared identities); summary.md gets `AppendIntegrationFix`'s section; no `-1` record; the outcome stays done.
  Flaky or pre-existing identities in the post-fix triage are recorded as today's triage section and warnings.
- Any other result: today's escalation unchanged (the pre-fix bisect result under the reserved `-1` record via `RecordIntegrationFailure`, `AppendIntegrationFailure`, no second bisect), plus `Fix` in the report and `AppendIntegrationFix`'s section.
  The stuck reason is `triageStuckReason`'s text plus the attempt's result and the refusal row's way forward: the pre-fix head and the fix commits, keep them or drop them with `git reset --hard <pre-fix head>`, fix the regression on the task branch, run the plan's `## verify:` and commit, then re-run `lyx webster run` (re-step the Webster row).
- `IntegrationFix.Result` is set in the same save.
Go never resets the branch: a failed attempt's commits stay on it.

Bound: this adds a spawn and commits to the task branch after the last batch, with no human.
It is bounded by the single attempt recorded before the spawn (a resumed run never spends a second one, and an attempt the run's end interrupted escalates rather than passing unchecked; `run --fresh` resets it), the `recovery_timeout_min` timeout, `checkFixCommits` with the plan-fingerprint compare, the post-fix triage deciding success, and the later loom review rows seeing the fix commits like any other branch commit.
Triage sees only failures, so a test the strand deleted or skipped reads as cleared; the prompt forbids it and summary.md names the commits and cleared identities for review.
Accepted residual, as for recover-batch: a process killed inside the strand's startup window leaves a strand whose GUID was never recorded.

Docs: extend `doc.go`'s integration-stage section with the attempt, its bounds and the reclaim; in `refusal-spec.md`'s webster table add a row "integration regression not fixed" (trigger: the run-exit integration stage finds a regression its one fix attempt did not clear, the attempt was already spent, or the attempt was interrupted by the run ending; class: correctness halt; way forward: done is demoted to stuck, and summary.md names the pre-fix head and the fix commits; keep them or drop them with `git reset --hard <pre-fix head>`, fix the regression on the task branch, run the plan's `## verify:` and commit, then re-run `lyx webster run` (re-step the Webster row)).

Tests in `integration_test.go`, with a fake `FixStarter` whose handle commits in the fixture worktree and writes the fix report:
- #332b Seen: a regression bisected to card 6; the strand commits a fix; the re-run triage passes; outcome done, no `-1` record, summary.md and the report record the fix commits and the cleared identity.
- The strand reports `FAILED`; the strand times out; the strand's commit leaves the regression: each escalates with the `-1` record, outcome stuck, and summary.md records the attempt.
- The refusal row's reaching test: the stuck reason of an unfixed attempt and of a spent one names the pre-fix head, each fix commit, `git reset --hard`, the verify-and-commit step and the `lyx webster run` re-run.
- A strand commit touching the plan directory, one touching `_lyx`, an uncommitted plan-file edit the strand leaves on disk, a dirty worktree, and HEAD moved past `head_sha` by a non-merge commit: each fails the attempt and escalates.
- A state with a spent `IntegrationFix` escalates with no `StartFix` call.
- A state with an in-flight `IntegrationFix` (empty `Result`) and an `OK` integration report ends stuck; summary.md names the commits after `PreFixHead`, and `Result` is set.
- A nil `FixStarter` over a regression returns the wiring error and leaves `IntegrationFix` unrecorded.
- Flaky-only and pre-existing-only verdicts make no `StartFix` call.
- A non-done Master outcome over a regression makes no `StartFix` call.
- `TestIntegrationStage_Regression_DemotesDone` and the other regression tests supply a fake starter that reports `FAILED`, keeping their escalation assertions.

**ImpactSummary:** the run-exit integration stage gains a spawn-and-verify step on regressions; RunDeps gains two fields that webstercli wires; escalation records gain the fix attempt.
