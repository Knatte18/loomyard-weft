# Card 10 — batten-resume-readopt

**Create:**
- `plan:internal/battencli#readoptSeams` -> `type readoptSeams struct`
- `plan:internal/battencli#hubReadoptSeams` -> `func hubReadoptSeams(location *lymcwd.Location) readoptSeams`
- `plan:internal/battencli#reconcileGoneRun` -> `func reconcileGoneRun(location *lymcwd.Location, slug string) (fabricengine.OriginReport, error)`
- `plan:internal/battencli#taskWorktreeGone` -> `func taskWorktreeGone(prime *lymcwd.Location, slug string) (bool, error)`
- `plan:internal/battencli#battenCLI.readoptGonePair` -> `func (c *battenCLI) readoptGonePair() error`
- `plan:internal/battencli#TestOwnerPreflight_GoneTaskWorktree` -> `func TestOwnerPreflight_GoneTaskWorktree(t *testing.T)`
- `plan:internal/battencli#TestBattenIntegration_GoneTaskWorktree` -> `func TestBattenIntegration_GoneTaskWorktree(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#Command`
- `internal/battencli#StartFirstPass`
- `internal/battencli#battenCLI.ownerPreflight`
- `internal/battencli#battenCLI.refuseContinuePass`
- `internal/battencli#TestBattenPreRun_ContinuePass`
- `internal/battencli#TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal`
- `internal/battencli#TestStartFirstPass`

**Uses:**
- `internal/battencli#reconcileRun`
- `internal/battencli#resumeRefusal`
- `internal/battencli#battenCLI.reconcileResume`
- `internal/battencli#spawnPairPush`
- `internal/battencli#taskWorktreeLocation`
- `internal/battencli#newFakeReceiver`
- `internal/battencli#fakeOwners`
- `plan:internal/fabricengine#PruneRegistrations`
- `plan:internal/fabricengine#ErrPairNotLive`
- `internal/fabricengine#NewTopology`

**Intent:** Decision record D1, batten's side: a `lym batten run` resume whose task worktree is gone judges origin, then re-adopts a live pair.

`battenCLI` gains a field `readopt readoptSeams`; `Command` sets it to `hubReadoptSeams(c.location)` for the run verb, and `StartFirstPass` sets it on the receiver its pre-checks run on, so adopt's green start and `lym beam create`, which reach `preflightRunOwner` through `startFirstPass`, take the same path.
A receiver whose `readopt.Add` is nil, every hand-built test receiver and the step verb's `Arm` receiver among them, keeps today's path: the resume reconcile, which refuses the absent pair through `originTargets`' error.
`readoptSeams` holds:
- `Gone func(slug string) (bool, error)`, `taskWorktreeGone` in production.
- `Reconcile func(slug string) (fabricengine.OriginReport, error)`, `reconcileGoneRun`: `reconcileRun`'s body with `ReconcileScope{Run: slug, PairMayBeAbsent: true}`.
- `Prune func() error`, `fabricengine.PruneRegistrations(location)`.
- `Add func(slug string) error`: `fabricengine.NewTopology(cfg).Add(location, slug, AddOptions{AdoptLocal: true, AdoptOnly: true})`, no `Parent` and no `Anchor`, with `cfg` loaded as the Worktree-Create row loads it.
- `SpawnPush func(slug string) error`: `taskWorktreeLocation` then `spawnPairPush`, as the Worktree-Create row spawns the new pair's push.

`taskWorktreeGone` reads true when nothing stands at `fabricengine.WorktreePath(prime, slug)` or a path there does not resolve through `lymcwd.ResolveWorktree`, unlike `taskWorktreePresent`, so a stray path reaches `Add`'s `*ErrTaskPathTaken` instead of failing the pre-run every pass; a stat error other than not-exist is returned.

`ownerPreflight`, for a found status whose `CurrentProducer` is neither Worktree-Create nor Worktree-Teardown, with `readopt.Add` set and `readopt.Gone` true, runs `readoptGonePair` in place of the resume reconcile, before the owner-seams branch:
1. `readopt.Reconcile`; an error refuses with its cause and `re-run lym batten run <slug>`; an unreachable origin refuses the pass with `origin is unreachable, so the gone pair cannot be judged; re-run lym batten run <slug> once origin answers` and writes no origin-owed marker; otherwise `resumeRefusal` over the report refuses an ended, stopped or owned-elsewhere entry as any resume does (the owned-elsewhere refusal is `*OwnedElsewhereError`).
2. `readopt.Prune`; an error refuses with its cause and the re-run.
3. `readopt.Add`; any error refuses the pass with `Add`'s text and the re-run, leaving the status as it is (card 12 sorts its permanent answers).
4. `readopt.SpawnPush`, a failure only warned, as the Worktree-Create row warns; then `reconcileResume`, the second reconcile, which now judges the pair's branches, claims the prime owner and resumes on its verdict.

`refuseContinuePass`, when `readopt.Gone` is set and reads true, refuses with the reason `its task worktree is gone`, so dobby owes a full pass and the next pass is a resume; a nil `readopt.Gone`, every hand-built receiver's, checks nothing, so the existing continuable rows keep passing.
A `Gone` error refuses with the reason naming it.

Bound: a pass may now re-create a task worktree with no operator step, and only over a live pair: `Add`'s adopt-only option re-creates no pair whose weft is archived or absent, whose task branch is gone locally and on origin, or over a stray records directory or task path; an entry origin reads as ended, stopped or owned elsewhere is refused before `Add` runs.

Tests:
- `TestOwnerPreflight_GoneTaskWorktree`, `tier0`, a table over `newFakeReceiver` with a status file at a row and a fake `readopt` recording its calls:
  - At Run-Shed, `Gone` true, active verdicts: calls run Reconcile, Prune, Add, SpawnPush, then the second reconcile (`c.reconcile`), in that order; nil error; status still `running`.
  - Owned elsewhere on the first reconcile: `*OwnedElsewhereError`, Add never called.
  - Ended and stopped first verdicts: the `resumeRefusal` texts, Add never called.
  - Unreachable first reconcile: the refusal names `once origin answers`, Add never called, no origin-owed marker.
  - First reconcile returning an error: the refusal names its cause and the re-run, Prune and Add never called.
  - Prune returning an error: the refusal names its cause and the re-run, Add never called.
  - Add returning a transport error, an error wrapping `fabricengine.ErrPairPushRunning`, and a `create worktree … failed` error: the pass refuses naming the re-run; status still `running`.
  - `Gone` false: today's `reconcileResume` only.
  - At Worktree-Create and at Worktree-Teardown with `Gone` true: the readopt seams are never called.
- `TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal` gains rows for `taskWorktreeGone`: no path reads gone; a plain file and an empty directory at the path read gone; a resolving worktree reads not gone.
- `TestBattenPreRun_ContinuePass` gains a row: a run otherwise continuable whose receiver's `readopt.Gone` reads true is refused with `its task worktree is gone`.
- `TestStartFirstPass` gains an assertion that the receiver it builds carries `readopt` seams.
- `TestBattenIntegration_GoneTaskWorktree`, `tier1`, over a `hubforge` hub with a pushed pair and the prime's batten status `running` at Run-Shed, driving `ownerPreflight` on a receiver with `hubReadoptSeams` whose `SpawnPush` runs `fabricengine.DetachedPush` synchronously:
  - Both worktree directories deleted, weft branch on origin unarchived: the pair is re-made, the status stays `running`, and the second reconcile's report has no stopped entry.
  - The task worktree alone deleted, the records worktree, weft and task branches standing: only the task worktree is re-made, wired to the standing records worktree.
  - A beam project's pair at a subpath anchor, its task worktree gone: re-made at the anchor its weft records.
  - A local weft branch origin lacks under a stale `refs/remotes/origin/<weft>`, and likewise a local task branch: the refs are dropped, the second reconcile reads no branch of the pair gone, and after the push origin holds the weft branch (D8's criteria).
  The stale-registration and prime-dirt criteria are reached at fabric level by card 4's `TestAdd_AdoptOnly` rows, and the prune's place in the order by the tier0 table.

**ImpactSummary:** Adds one seam field to `battenCLI`, set only by the run verb's `Command` and `StartFirstPass`; the step verb and every hand-built receiver keep today's path, and the continue-pass refusal gains one reason that dobby already reads as a full pass owed.

**Commit:** `10: A batten resume re-adopts a live pair whose task worktree is gone`
