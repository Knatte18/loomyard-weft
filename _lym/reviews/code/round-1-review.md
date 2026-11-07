---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hubreconcile/walk.go:295
    origin: lens:code-tests
    summary: the swallow-and-log path of a failed overview render has no test
    fix: In internal/hubreconcile/ensure_integration_test.go, add a third row to TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview whose seed commits a heddle.json on the board that fails to load. The row expects Ensure to return nil, HEDDLE.md to stay as it was, and the walk's other config files to still be committed. Keep the failed-board-write restore steps for the first two rows only.
  - id: F2
    severity: LOW
    class: design
    location: internal/hubreconcile/walk.go:264
    origin: lens:code-tests
    summary: no row starts from an overview that already matches, so the identical render commits nothing half of the Goal is unexercised
    fix: In internal/hubreconcile/ensure_integration_test.go, add a case where the board's HEDDLE.md already holds what heddleengine.RenderOverview would write and the walk has a reason to commit a config file. Assert the walk's commit does not name HEDDLE.md and that RenderOverview afterwards reports no change. Fold it into TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview as a row.
---

### [MEDIUM] Failed-render path is untested

**Location:** `internal/hubreconcile/walk.go:295`, `internal/hubreconcile/ensure_integration_test.go:909`

**Issue:** `renderHeddleOverview` logs a failed render at Warn, returns false and lets the walk go on.
The change description and `internal/hubreconcile/doc.go` both state this as a contract.
No test runs the walk over a `heddle.json` that fails to load.
A regression that returned the error would fail every start verb's config reconcile, and nothing would catch it.
The new test's two rows seed only a missing and a stale overview.

### [LOW] No row for an up-to-date overview

**Location:** `internal/hubreconcile/walk.go:264`, `internal/hubreconcile/ensure_integration_test.go:909`

**Issue:** The decision record asks that a render that would write identical bytes commits nothing.
Both rows of the new test begin with a missing or stale overview, so the `changed == false` branch of `walk.go:264` has no row.
The staging layer drops an unchanged path anyway, so an always-append mutant would pass.
The Goal's second half is still behavior the diff adds with nothing pinning it.

## Rejected

- Lens code-correctness F1 (an overview left rendered but uncommitted by an earlier `lym heddle set` is not committed by the walk when the board has no upstream): that file belongs to the set's own board sync, which commits it, and the Goal asks only for a changed render to be committed.
- Lens code-correctness F2 (a failed render still writes the stamp, so a retry waits for the next build or heddle write): the change description records this as a deliberate trade-off, and the Warn carries a `way_forward` naming the next heddle write.
- Lens code-tests F3 (no test asserts that a skipped walk leaves `HEDDLE.md` untouched): the render sits inside the `Write` callback that `PullThenCommitWritten` calls only after a successful pull, so the behavior holds by structure and no code of the diff branches on it.
- Lens code-affected-set F1 (`arrangeUnpushedHubWide` may now fail on a second changed file): I ran `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`, `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` and the new test through `lym gate test --tier 1 ./internal/hubreconcile`, and all pass, so the fixture board already holds a current overview.

Settled against the focus directive and the Goal, with no defect found:

- The overview joins `written` the way config files do, so a changed file is committed and an identical render adds nothing, including when no config file changed.
- The render sits inside the `Write` callback, after the pull and under the Bolt write lock, so a skipped pull renders nothing, and every failure after it restores `HEDDLE.md` through the snapshot taken at `walk.go:228`.
- `RenderOverview` writes atomically after it has read the layers, so an error leaves the file as it was, and it takes no lock of its own, so there is no re-entrancy against the walk's lock.
- The Warn call carries a `way_forward` key like the package's other Warn calls.
- `lym heddle set` and `lym heddle remove` are unchanged, and the diff touches only the walk, its docs, the import allowlist and `PATTERN-heddle-instructions`.
- The `heddlecli` test edit pins that a save leaves the overview for the board sync.
- The commits carry no `Pattern-Departure:` trailer and none is needed.
