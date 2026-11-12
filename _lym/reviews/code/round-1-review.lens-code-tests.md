```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none (lens code-tests).

**Per behaviour, the test that pins it**

- **Start rule on no exit record, a moved HEAD or queued work.** `TestTestMapStep/the start rule` (`internal/dobbyengine/testmap_test.go`). Pinned.
- **One-slot hub: queued work alone starts no pass, and slots are not read when nothing is queued.** The same table. Pinned.
- **`test_map_upkeep: false` starts no pass.** The same table. Pinned.
- **A registered pass, including a hand-started one, holds the start.** The same table (`running`). Pinned.
- **The dobby's own unregistered live child holds the start, and the tick that reads its exit starts none.** The "live child" subtest. Pinned.
- **Failed exit logged once per episode, clean exit ends the episode.** The "failed exit is logged once per episode" subtest. Pinned.
- **Failing start and failing reads return a wrapped error and start nothing.** The "failing start or read" table. Pinned.
- **A failing upkeep step leaves the rest of the tick running.** `TestRun_AFailingTestMapStartLeavesTheRestOfTheTick`. Pinned.
- **An anchor move that stands holds the upkeep step.** `TestRun_NoPassWhileAMoveStands` (`loop_move_test.go`) and `runAnAdoptHold` in `handover_test.go`. Pinned.
- **`startTestMap` passes `--upkeep`, runs in the anchor, delivers the exit once, carries the code and output tail.** `TestStartTestMap`. Pinned.
- **The prime fills the five upkeep seams and a beam fills none.** `TestHubDobbyDeps_PrimeTidyShuttle`. Pinned. The real closures that read the exit record, HEAD and slots are not run by any test; this is the seam-wiring check only.
- **Per-layer `MigrateLayer` hook: runs in the base, worktree and local layers, writes no file, and a failure is `ErrInvalid` naming the layer.** `TestLoad_MigrateLayer` (`layers_test.go`). Pinned.
- **That the hook runs before the merge and before `DropRetired`.** The same test, since the retired key `old` is only read if the hook ran first. Pinned.
- **`MigrateConfig`: `test_map_every: 0` becomes `test_map_upkeep: false`, any other value is dropped, comment handling.** `TestMigrateConfig` (`config_test.go:122-129`). Pinned.
- **Beam config load: unreconciled worktree and local layers read `test_map_every: 0` as off, and `test_map_every`/`test_map_batch` load as retired keys.** `TestLoadConfig` (`config_test.go:66-68`). Pinned.
- **beam's `Migrate` is wired in `configreg`.** `TestPrepareRunSettings`, the "retired test_map_every of 0" row. Pinned.
- **Negative-value refusal and the removed `test_map_batch < 1` refusal.** `TestLoadConfig` pins this. The `config_test.go` diff was not read in full, so this is assumed from the harness rows rather than checked.

**Findings**

1. **MEDIUM, scope.** `internal/fabriccli/anchordeps.go:482`. `dobbyPassInFlight` now spares `RoleTestMap` (D11, and an acceptance criterion: "an anchor move that finds a pass running ends it and does not wait"). The diff changes no test in `internal/fabriccli`, and a grep of `internal/fabriccli/*_test.go` for `dobbyPassInFlight` and `RoleTestMap` returned nothing. Reverting the one-line change would break no test. Fix: add a row to the in-flight test, or a new one if none exists, with a registered `RoleTestMap` child of the dobby and no run lock, asserting empty "in flight". Also assert that an unregistered child of the dobby still counts.

2. **LOW, scope.** `internal/dobbyengine/state.go`. D8 and the constraints say the retired landing count "stays readable in an old dobby state record" (PATTERN-tolerant-records). The diff removes the `LandingsSinceTestMap` field. No test in the diff reads a state JSON holding `landings_since_test_map` and expects no error. Go's JSON decoder tolerates the unknown key, so this is a regression guard rather than a likely break. Fix: add a `ReadState` row or assertion over a record holding `landings_since_test_map`, if `state_test.go` has no such row. I did not check `state_test.go`, and no tool call was spent on it.

3. **LOW, scope.** `internal/configreg` and `hubreconcile`. The acceptance criteria require `test_map_every: 0` to read as off after the reconcile walk, and in the prime's `_lym/config/beam.yaml` and the local layer under `ReconcileAll`. The diff pins only `Module.Migrate` through `PrepareRunSettings` and the in-memory load. No test in the diff drives `ReconcileHubWideAt` or `ReconcileAll` over a hub base file or layer holding `test_map_every: 0`. The walk-order claim, that `Migrate` runs before `pruneSparse`, is therefore pinned nowhere. Fix: add a case to the existing reconcile test table (not a new test) that asserts a hub base file holding `test_map_every: 0` ends as `test_map_upkeep: false`. I did not look for an existing reconcile test that already covers this, and made no tool call for it.

4. **LOW, scope.** `internal/gatecli`. D8's "`--upkeep` call whose registration fails exits before it plans" is a changed behaviour of the dobby's counterpart. It sits in `testmap.go` and its tests belong to the earlier testmap-upkeep-runs part, outside this diff. I note it only because `TestStartTestMap` uses a stand-in, so the real exit is not exercised here.

**Test economy**

- No duplicates found. `TestStartTestMap` and `TestHubDobbyDeps_PrimeTidyShuttle` are listed "not judged, unclassifiable", and each pins a distinct boundary: the real process spawn and the seam wiring.
- `TestLoad_MigrateLayer` is unique per the report, and `TestMigrateConfig` and `TestLoadConfig` cover different layers of the same feature.

**Tool calls**

1. One `git diff` over the changed test files, plus `sed` of `testmap_test.go` lines 200-330 and a grep of `internal/fabriccli/*_test.go`. My context lacked the test diffs and the second half of `testmap_test.go`. This was the only call I made.
