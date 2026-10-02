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
- `plan:internal/websterengine#RenderIntegrationFixPrompt`
- `plan:internal/websterengine#IntegrationFixState`
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
1. Leased: reload state. A non-nil `IntegrationFix` means the attempt is spent: escalate without spawning, with result `spent`.
   Otherwise record `IntegrationFix` with `PreFixHead` (HEAD now) and `SpawnedAt`, and save.
2. Unleased: render the prompt with `RenderIntegrationFixPrompt` (the regressions with their tails, `plan.Verify`, the localized card's file as the hint when bisect named one, a fix report path under the reports directory, the plan directory) and start the strand at `RoleRecovery`'s model with `recovery_timeout_min` as its timeout and the report as its output file.
   A start error is a failed attempt.
3. Leased: record `StrandGUID` and save.
4. Unleased: wait on the handle; then remove the strand if live and its run dir, as recover-batch does.
   A timeout or dead outcome, a missing or malformed report, or `status: FAILED` is a failed attempt.
5. Unleased: `checkFixCommits(worktree, PreFixHead, report head_sha, planDir, ParentBranch)`; an error is a failed attempt with result `refused`.
6. Unleased: run the verify once at HEAD into a dedicated scratch log; on red, triage it with `triageIntegrationFailure` against the same start commits.
   No regression in the result is success: the post-fix triage decides, never the strand's report.

`recordTriageResult` records the result in phase 3:
- Success: the report carries the pre-fix failures and triage plus `Fix` (result `fixed`, commits, cleared identities); summary.md gets `AppendIntegrationFix`'s section; no `-1` record; the outcome stays done.
  Flaky or pre-existing identities in the post-fix triage are recorded as today's triage section and warnings.
- Any other result: today's escalation unchanged (the pre-fix bisect result under the reserved `-1` record via `RecordIntegrationFailure`, `AppendIntegrationFailure`, no second bisect), plus `Fix` in the report and `AppendIntegrationFix`'s section.
  The stuck reason is `triageStuckReason`'s text plus the attempt's result and a way forward naming the pre-fix head and the fix commits.
- `IntegrationFix.Result` is set in the same save.
Go never resets the branch: a failed attempt's commits stay on it.

A nil `FixStarter` when a regression reaches step 2 is a wiring-guard error, never a skipped attempt.

Bound: this adds a spawn and commits to the task branch after the last batch, with no human.
It is bounded by the single attempt recorded before the spawn (a resumed run never spends a second one; `run --fresh` resets it), the `recovery_timeout_min` timeout, `checkFixCommits`, the post-fix triage deciding success, and the later loom review rows seeing the fix commits like any other branch commit.
Triage sees only failures, so a test the strand deleted or skipped reads as cleared; the prompt forbids it and summary.md names the commits and cleared identities for review.
Accepted residual, as for recover-batch: a process killed inside the strand's startup window leaves a strand whose GUID was never recorded.

Docs: extend `doc.go`'s integration-stage section with the attempt, its bounds and the reclaim; in `refusal-spec.md`'s webster table add a row "integration regression not fixed" (trigger: the run-exit integration stage finds a regression its one fix attempt did not clear, or the attempt was already spent; class: correctness halt; way forward: done is demoted to stuck, and summary.md names the pre-fix head and the fix commits; keep them or drop them with `git reset --hard <pre-fix head>`, fix the regression on the task branch, run the plan's `## verify:` and commit).

Tests in `integration_test.go`, with a fake `FixStarter` whose handle commits in the fixture worktree and writes the fix report:
- #332b Seen: a regression bisected to card 6; the strand commits a fix; the re-run triage passes; outcome done, no `-1` record, summary.md and the report record the fix commits and the cleared identity.
- The strand reports `FAILED`; the strand times out; the strand's commit leaves the regression: each escalates with the `-1` record, outcome stuck, and summary.md records the attempt.
- A strand commit touching the plan directory, one touching `_lyx`, and HEAD moved past `head_sha` by a non-merge commit: each fails the attempt and escalates.
- A state with a spent `IntegrationFix` escalates with no `StartFix` call.
- Flaky-only and pre-existing-only verdicts make no `StartFix` call.
- A non-done Master outcome over a regression makes no `StartFix` call.
- `TestIntegrationStage_Regression_DemotesDone` and the other regression tests supply a fake starter that reports `FAILED`, keeping their escalation assertions.

**ImpactSummary:** the run-exit integration stage gains a spawn-and-verify step on regressions; RunDeps gains two fields that webstercli wires; escalation records gain the fix attempt.
