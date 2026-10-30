```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each function and branch the diff changes against a concrete input.

**Per-case list**

- `landBeam`, nil `CanLand` or empty slug (prime's dobby): returns at once, no read. Correct.
- `landBeam`, `CanLand` error: logged through `landRead.Fail`, state untouched, next tick retries. Correct.
- `landBeam`, reading does not pass with `Land` nil: no write. Correct.
- `landBeam`, reading does not pass with `Land` set: state cleared, which re-arms the step. Correct.
- `landBeam`, reading passes with `Land` set: no action. Correct.
- `landBeam`, passes with `ParentOwner` set: posts the adopt notice and records the episode once the post goes out. A failed post leaves the episode open, so the next tick retries. Correct.
- `landBeam`, `Land()` returns nil: logs, records the episode, no notice. Correct.
- `landBeam`, `Land()` refused because the run is not yet at Beam-Await: the refusal is posted and the episode recorded. A later tick never retries while the reading keeps passing. **Not correct, see F1.**
- `landBeam`, `Land()` succeeds but `recordLand` fails: warns and acts again next tick. This is a harmless rewrite of the same request while Beam-Await has not yet consumed it. Correct enough.
- `postLandNotice`, `Notice` error: `landPost.Fail`, episode left open, retried next tick. Correct.
- `recordLand`: `UpdateState` stores a `LandState` with the current time. Correct.
- `beamLandReading`, unflagged beam: `Ready` is false, so it returns an empty reading and a nil error before `ReadBeamLandBlockers` runs. Correct.
- `beamLandReading`, `Ready` returns an error: the error is returned and the dobby logs an unreadable state. Correct.
- `deps.Land`: runs `land(hubLandDeps(prime), geom.Slug)`, the same sequence `lym beam land` uses, so the same land request is written. Refusals are returned, not swallowed. Correct.
- `land()` ordering: entry check, run check, owner checks, ready check, work-remaining refusal. The unflagged case gets its own message naming `lym beam ready <slug>`. A `Ready` error is wrapped with a way forward. Correct.
- `markReady`, unflagged beam: reads `Ready` false, calls `MarkReady` with `SetAt` in UTC, returns the record path. Correct.
- `markReady`, already flagged: returns `already: true`, writes nothing. Correct.
- `markReady`, non-landable entry: refused with the `lym beam ready:` prefix by `judgeLandEntry`. Correct.
- `markReady`, `MarkReady` fails on the commit after the record was written: **not correct, see F2.**
- `judgeLandEntry` with a `verb` parameter: every message uses the verb. The `fabric reconcile` way forward becomes `%s %s`, so it reads `lym beam ready <slug>` for ready. Correct.
- `onHubLandDeps`: error wrapping is the same as before with `verb` substituted. Correct.
- `hubLandDeps.Ready` / `MarkReady`: `beamOf` errors are propagated. The commit pathspec is `LandingDirRel`, which stages the whole landing directory. That matches what the Describe row does. Correct.
- `ReadyRecord` / `WriteReady`: creates the lock directory, then `state.WriteJSON`. Correct.
- `ReadyFor`, absent record: false, nil. Correct.
- `ReadyFor`, record naming another slug: false. Correct.
- `ReadyFor`, corrupt record: error, which becomes an unreadable-state log on the dobby and a refusal in `land`. Correct.
- `BeamReadyPath` / `BeamReadyLockPath`: `_lym/landing/beam-ready.json` and `.lym/landing/beam-ready.json.lock`, both anchor-relative. Correct.
- `pawlRecheck`: `location` is the beam's own worktree. `Recheck` is built from `location, slug` at `pawlwiring.go:133` and the landing directory is anchor-relative, so `ReadyFor` reads the beam's own record. Correct.
- `pawlRecheckLines`, flagged beam with nothing else wrong: no line. Unflagged: the ready line comes last. Correct.
- State rename `LandNotice` to `Land` (JSON key `land_notice` to `land`): a state file written by an older binary is read tolerantly, so the old key is ignored. A beam already in a passing episode gets one landing step after upgrade. That is acceptable, since the old notice meant "can land" and the new gate also needs the flag.

**Findings**

F1: MEDIUM, design. `internal/dobbyengine/land.go` (the `d.deps.Land()` error branch, with `postLandNotice` recording the episode) together with `internal/beamcli/land.go` (`judgeLandRun`).
- A transient refusal fixes the episode for good. A beam flagged before its pawl run reaches Beam-Await, such as one flagged right after `lym beam create`, makes `Land()` refuse because the run is not waiting at Beam-Await.
- The dobby posts the notice and records the episode. The reading keeps passing, so the step is never retried once the run reaches Beam-Await, and the beam sits flagged without landing until someone runs `lym beam land`.
- This contradicts the entry's design: dobby starts the landing as soon as the flag is set.
- The fix is to not record the episode on a `Land` refusal that is only a run-state wait. For example, retry the step on later ticks and post the notice once per episode, or have `Land` report a "not yet" result separately from a hard refusal.
- Evidence: `land.go` lines 57-58, where the refusal goes straight to `postLandNotice`, which calls `recordLand`; `beamcli/land.go:54`, where `judgeLandRun` runs inside `land`.

F2: LOW, design. `internal/beamcli/deps.go`, `MarkReady`.
- The record is written before the commit. If `CommitAnchoredPaths` fails, `markReady` returns an error telling the user to run `lym beam ready <slug>` again.
- The retry reads `Ready` as true, reports `already: true` and never commits. The flag then sits uncommitted in `_lym`, and the whole landing directory is left unstaged.
- Land reads the file, so the beam lands locally, but the durability the design relies on is lost.
- The fix is to commit the landing directory even when the record already exists, or to remove the record when the commit fails.
- Evidence: `deps.go` `MarkReady` lines 125-133, `land.go` `markReady` lines 99-109.

**Tool calls**
1. Read `internal/loomcli/pawlwiring.go` 150-240 and grepped the `pawlRecheck(` callers. My context lacked which worktree `location` is at the Beam-Await re-check.
