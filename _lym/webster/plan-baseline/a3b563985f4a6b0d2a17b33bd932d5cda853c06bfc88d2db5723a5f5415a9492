# Card 20 — hand-pass-catch-up

**Create:**
- `plan:internal/dobbyengine#PassExitOriginWait` -> `const PassExitOriginWait = 80`
- `plan:internal/battencli#HubCatchUp` -> `func HubCatchUp(prime *lymcwd.Location, runs func() ([]string, error)) dobbyengine.CatchUpDeps`
- `plan:internal/battencli#DobbyPairSlugs` -> `func DobbyPairSlugs(prime *lymcwd.Location, geom dobbyengine.Geometry, parents *DobbyParents) ([]string, error)`
- `plan:internal/battencli#battenCLI.catchUpRun` -> `func (c *battenCLI) catchUpRun() error`
- `plan:internal/battencli#TestHubCatchUp` -> `func TestHubCatchUp(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.reconcileResume`
- `internal/battencli#battenCLI.driveRun`
- `internal/battencli#battenCLI.refuseContinuePass`
- `internal/battencli#battenCLI.claimBoardEntry`
- `internal/battencli#reconcileGoneRun`
- `internal/battencli#DobbyRuns`
- `internal/battencli#passExitCode`
- `internal/battencli#TestPassExitCode`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/battencli#TestBattenPreRun_ContinuePass`
- `internal/battencli#TestOwnerPreflight_GoneTaskWorktree`
- `internal/battencli#TestOwnerPreflight_TeardownResume`
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine#daemon.spawn`
- `internal/dobbyengine#daemon.passExited`
- `internal/dobbyengine#TestPlanPass`
- `internal/dobbyengine#TestDrivePasses_ExitHandling`
- `internal/dobbyengine/doc.go`

**Delete:**
- `internal/battencli#OriginOwed`
- `internal/battencli#originOwed`
- `internal/battencli#battenCLI.markOriginOwed`
- `internal/battencli#battenCLI.clearOriginOwed`
- `internal/battencli/originowed_test.go`
- `internal/dobbyengine#daemon.originOwed`
- `internal/dobbyengine#passFullNoReopen`
- `internal/shedrun#OriginOwedMarker`
- `internal/shedrun#originOwedMarkerFileName`

**Uses:**
- `plan:internal/dobbyengine#CatchUp`
- `plan:internal/pairteardown#HandOverOwnedElsewhere`
- `internal/pairteardown#PushRun`
- `internal/pairteardown#ReadRun`
- `plan:internal/tendercli#PauseIntentEntries`
- `plan:internal/boardengine#EnsureIdentity`
- `plan:internal/boardengine#RegisterMachine`
- `internal/shedrun/doc.go`

**Intent:** Wire the catch-up for a run's own pass, and retire the origin-owed marker it replaces (D4: "a hand pass, whose catch-up runs over its own run only"; "The origin-owed marker ... retires").

`HubCatchUp(prime, runs)` fills `dobbyengine.CatchUpDeps` over the hub:
`PullBolt` is the Bolt's `Pull` (fetch, then replay of local display commits), a `fetch_failed` skip mapping to `ErrOriginUnreachable` and any other skip (`diverged`, `dirty`) to an error naming the skip's detail and its way forward;
`Register` is `boardengine.EnsureIdentity` then `boardengine.RegisterMachine`, `ErrOwnerOriginUnreachable` mapping to `ErrOriginUnreachable`;
`FetchRun` is `fabricengine.FetchPairTips` of the run's pair (or the prime's records branch for a run with no pair here), mapping a transport failure to `ErrOriginUnreachable`;
`ReadRun` reads `pairteardown.ReadRun` for the owner and `OwnedElsewhere`, and `Ahead` from whether the prime's records side or the pair's records or code side holds commits its upstream lacks;
`PushOwed` is `PushRun` of the prime's records and then the pair, `ErrHandedOver` answering nil;
`HandOver` is `pairteardown.HandOverOwnedElsewhere`; `ApplyPauseIntents` is `tendercli.PauseIntentEntries`.
A pair directory that stands but is not a registered linked worktree, as `Teardown.TaskWorktreeGone` reads it, is a run with no pair here for `FetchRun`, `Ahead` and `PushOwed`, so a leftover directory fails no catch-up and no git command runs in it.
`DobbyPairSlugs` lists the slugs of the runs whose parent is the dobby's worktree and whose pair stands here (the pair's worktree path exists, the predicate `RunView.PairPresent` reads), for the dobby's scope in card 21;
for the prime's dobby it also lists every local pair whose parent beam reads owned elsewhere, since `EnsureLive` starts no dobby for such a beam, so each local pair is caught up by some dobby (D4 step 2: "each local pair's branches").

A pass that is not a continue pass runs `catchUpRun` before its resume reconcile, through a `catchUp` seam on `battenCLI` that `wire` fills with `CatchUp(HubCatchUp(prime, [slug]))`; a hand-built receiver's nil seam skips it.
Its scope is the pass's own slug alone, so a hand pass fetches only its own run's branches.
An unreachable origin refuses the pass with nothing acted on: `batten run: origin is unreachable, so the catch-up acted on no run; way forward: run lym batten run <slug> again once origin answers`, exit `dobbyengine.PassExitOriginWait`.
A registration refusal refuses naming the refusal's own way forward, with the same exit.
Any other failure refuses naming the catch-up's error and the way forward: fix the cause it names, then run `lym batten run <slug>` again; the same exit.
A run the catch-up handed over exits with `PassExitOwnedElsewhere`.
Bound: a dobby's full pass is not a continue pass, so it runs the catch-up of its own slug too; it fetches only that run's branches, beside the run-scope reconcile the full pass already runs.
That pass's catch-up can run beside the dobby's own catch-up over the same pair, which is safe:
pushes go under the push lock;
pair removal takes the teardown lock, so a second hand-over meets a busy lock and retries, or finds the pair gone;
and a pass whose push or hand-over loses that race fails it, so the pass refuses with the catch-up's error and acts on no run.

The marker retires: `reconcileResume`'s unreachable branch is now reached only when origin is lost between the catch-up and the reconcile; it logs and resumes with no marker, since the catch-up before it already read origin.
`driveRun` clears nothing; `refuseContinuePass` loses its marker clause; `DobbyRuns` stops reading it; `battenCLI` loses `originReached`;
`OriginOwed`, `originOwed`, `markOriginOwed`, `clearOriginOwed`, `shedrun.OriginOwedMarker` and its file name are deleted, with `originowed_test.go`.
In `dobbyengine`, `RunView.OriginOwed`, the `passFullNoReopen` kind and `daemon.originOwed` go: `planPass` gives a plain full pass, `spawn` passes no `--no-reopen`, and `passExited` clears an owed full pass on any full pass exiting 0.
`RunView.Held`'s doc names the `bolt-hold` entries card 12 counts, which the dobby lifts itself, beside the adopt entries only a hand lifts.
`passExited` reads `PassExitOriginWait` as no failing pass; card 21 makes it owe the dobby's catch-up.
`passExitCode` maps the catch-up's refusals to their exits.
The comments of `claimBoardEntry` and `reconcileGoneRun` stop naming the marker.
No code path reads or writes the origin-owed marker.

Tests:
`TestRunCmd_ResumeDispositions` gains the catch-up rows over a fake `catchUp`: unreachable refuses with the exit and the re-run; a registration refusal passes its way forward; any other failure names the error and the re-run; a hand-over exits owned elsewhere; a continue pass runs no catch-up; a clean catch-up goes on to the reconcile.
It drops its marker rows, and the unreachable resume row asserts the resume with no marker.
`TestHubCatchUp`, tier1, over a hub fixture with one local pair, reaches the production seams through `CatchUp(HubCatchUp(prime, [slug]))`: a pair whose records side holds a commit origin lacks has it pushed and nothing reset; a scope of one slug fetches only that pair's branches; a pair directory left standing with no worktree registration is read as no pair here and the catch-up succeeds; with origin unreachable the call returns `ErrOriginUnreachable` and pushes nothing.
`TestPassExitCode` gains the new exit.
`TestBattenPreRun_ContinuePass`, `TestOwnerPreflight_GoneTaskWorktree` and `TestOwnerPreflight_TeardownResume` drop their marker rows and assertions.
`TestPlanPass` drops its two marker rows, and `TestDrivePasses_ExitHandling` drops its marker subtest and gains a `PassExitOriginWait` exit counted as no failure.
`dobbyengine/doc.go` drops the marker and names the new exit.

**ImpactSummary:** Retires one marker file read in three places and written in one, adds the pass's catch-up with one new exit code; spawnPass's --no-reopen and every other pass rule are unchanged.
