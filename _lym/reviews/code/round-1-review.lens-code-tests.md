```yaml
lens: code-tests
outcome: reported
```

**Lens:** code-tests. For each behaviour the diff adds or changes, the test row that would fail if it broke, or none.

| Behaviour | Pinning test | Verdict |
|---|---|---|
| A missing overview is rendered and joins the walk's board commit | `TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview`, row "missing overview", checks `log -1 --name-only` for `HEDDLE.md` (`ensure_integration_test.go:951`) | pinned |
| A stale overview is renewed and committed | Same test, row "stale overview", plus `RenderOverview` returning `(false, nil)` afterwards and a clean status (`:955-960`) | pinned |
| A failed board write restores the overview's bytes, or its absence, and leaves the board status unchanged | Same test, both rows, via `EnsureFailingAfterHubWideWrite` (`:939-946`) | pinned; dropping the `snapshotFile` call in `walk.go:228` would fail both rows |
| A render whose bytes are identical commits nothing | None | no row (finding 2) |
| A failed render is logged at Warn, leaves the file as it was, and the config reconcile goes on | None | no row (finding 1) |
| A skipped walk (board cannot be brought up to date) renders nothing | None in this diff | no row (finding 3) |
| `heddle set` and `heddle remove` keep rendering the overview, left for the board sync to commit | `TestHeddleCLI`, edited assertion expecting `" M HEDDLE.md"` (`cli_integration_test.go:505`), and the existing overview-content check (`:230-236`) | pinned |
| `hubreconcile` imports `heddleengine` and nothing else new | `TestToldGeometryInvariant_AllowlistOnly` (stale-allowlist check) | pinned |
| `snapshotFile` extracted from `snapshotConfig` with unchanged behaviour | Existing restore tests, e.g. `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` | pinned by existing tests |

**Findings**

1. **MEDIUM, design** — `walk.go:295-303` (`renderHeddleOverview`).
   - No test covers the swallow-and-log path. `summary.md` and `doc.go` both claim that a failed render leaves the file as it was and that the walk goes on.
   - Nothing runs the walk over a `heddle.json` that fails to load, so a regression that returned the error would fail every start verb's config and no test would catch it.
   - Evidence: the new test's only seeds are a missing and a stale overview (`ensure_integration_test.go:918-924`), and a search of the tests found no other `heddle.json` fixture in `hubreconcile`.
   - Fix: add a third row to the existing table-driven test, seeding an unloadable `heddle.json`. The row should expect `Ensure` to return nil, the overview to stay as it was, and the config files to still commit.

2. **LOW, design** — `walk.go:264` (`changed == false` adds nothing to `written`).
   - No row starts from an overview that already matches. Both rows begin missing or stale, so the "identical render commits nothing" half of the Goal is never exercised.
   - A mutant that always appended `HEDDLE.md` would pass, because `stageablePaths` and `StageAndCommit` drop an unchanged path anyway.
   - The only observable effect is whether a walk with no other change makes an empty or extra commit.
   - Fix: add a row with a current overview and no other reconcile change, asserting that no commit is made.

3. **LOW, design** — `walk.go:279-281` and `:222-265`.
   - The summary and the focus claim a skipped walk renders nothing, but no test asserts that `HEDDLE.md` is untouched on a pull skip.
   - The behaviour holds structurally, since the render sits inside the `Write` callback that runs only after the pull succeeds.
   - Fix: add an assertion to an existing skip test, such as `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`, rather than a new test.

**Test economy:** no duplicate. The new test covers a behaviour no existing test covers, since the worktree restore tests do not touch the board overview. It has no per-helper test and no assertion on static prose.

**Tool calls:** none. The inherited context already held everything the lens needed.
