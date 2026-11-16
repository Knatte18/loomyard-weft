```yaml
lens: code-tests
outcome: reported
```

Task: run the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if the behaviour broke.

**Behaviour → pinning test**
1. A `testmap` call from a non-prime worktree builds at that worktree's HEAD and writes under its own `.lym`: `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` (`testmap_integration_test.go:668`).
2. A non-prime call leaves the hub's shared maps untouched: the same test (`testmap_integration_test.go:671`).
3. A call from the prime still reads and writes the shared maps: `TestGateTestMap`, which reads `mapsDir := testcover.Dir(fabricengine.BoltDir(...))` (`testmap_integration_test.go:760`, `787`).
4. The prime's records sibling acts in the prime: row `FromThePrimesRecordsSiblingActsInThePrime` (unchanged).
5. The refusal outside every hub has new way-forward text: row `OutsideEveryHub` (`testmap_integration_test.go:~839`).
6. `lym gate redundant` from a non-prime worktree reads that worktree's maps: `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` checks `map_commit == head`.
7. `lym gate redundant` from the prime reads the shared maps: `redundant_integration_test.go:38`, `71`.
8. The maps directory sits under the worktree's anchor, not its root (`coverageMapsDir` uses `location.AnchorPath()`): none.
9. `redundant`'s new rebuild way forward ("build it with lym gate testmap %s from this worktree"): none, because the error-path test checks only a substring.
10. The build lock is per worktree (`testMapBuildLockPath(r.mapsDir, …)`): none directly. It follows mechanically from `mapsDir`, which rows 1 and 3 pin, so I file no finding for it.
11. The `coverageMapsDir` error path from `hubgeom.IsPrime`, in both `testmap` and `redundant`: none. It is an internal failure with no reachable fixture, so I file no finding.
12. Removing the `OutsideThePrime` and `FromAPairsRecordsSibling` rows: the behaviour is removed, so this is correct. Nothing covers a call from a pair's records sibling now that it builds the pair's own maps. That path goes through the same `resolveHub`/`CodeLocation` as row 4, so I file no finding for it.

**Findings**

- **LOW**, class `design`, location `internal/gatecli/testmap_integration_test.go:641` (`TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`)
  - Summary: the only non-prime test uses `Anchor: "."`, so `AnchorPath()` equals `WorktreePath()`. Changing `coverageMapsDir` to `testcover.Dir(location.WorktreePath())` would still pass, so nothing pins the anchor choice the doc claims ("Those are `testcover.Dir` of its anchor").
  - Evidence: `testmap_integration_test.go:641`, and the comment at `:667` ("The hub's anchor is the worktree root"); `testmap.go:313`; hubforge supports a non-root anchor (`test_integration_test.go:2400`, `Anchor: "backend"`).
  - Fix: run that test, or a row of it, on a hub whose anchor is a subdirectory, and assert the map lies under `<worktree>/<anchor>/.lym/testcover`.
- **LOW**, class `design`, location `internal/gatecli/redundant.go:99` (the `rebuild` way forward)
  - Summary: the way forward changed from "from the hub's prime worktree" to "from this worktree", but `redundant_integration_test.go:123` checks only `strings.Contains(envelope.Error, "way forward")`. No row would fail if the old prime-pointing text came back.
  - Evidence: `redundant_integration_test.go:123`.
  - Fix: assert the exact way-forward text in the no-map row.

Tool calls:
1. grep of `redundant_integration_test.go` for way-forward assertions and of the gatecli tests for `Anchor:` shapes. My context showed neither how the redundant error rows assert the message nor whether any fixture uses a non-root anchor.
