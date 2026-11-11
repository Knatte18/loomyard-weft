---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/beamcli/deps.go:242
    summary: the hubLandDeps HeldSlugs wiring, which unions run directories and task branch slugs, has no test that reaches it
    fix: add a TestHubLandDeps_HeldSlugs case in internal/beamcli/deps_integration_test.go beside the other hubLandDeps tests (tiergate.Require(t, 1), a real hub pair); create a run directory under the prime's shed directory and a task branch of another slug, call hubLandDeps(prime).HeldSlugs(), and assert the result holds both slugs.
---

### [MEDIUM] HeldSlugs wiring in hubLandDeps is untested

**Location:** internal/beamcli/deps.go:242
**Issue:** `TestMergeParent` fakes `landWorld.HeldSlugs`, and `TestRunIDs` covers only the run-directory reader.
No test calls the real closure, so dropping the task-branch half, passing a non-empty prefix to `fabricengine.TaskBranchSlugs`, or swallowing an error from either source would fail nothing.
The closure is behaviour the diff adds, and the second half of the Goal (a leftover task branch) depends on it.

## Rejected

- Held run directory and held task branch lack separate `TestMergeParent` rows: `freeMergeTaskSlug` takes one merged list, so the row `CompletedRunAndLeftoverBranchSkipped` covers both.
- No `TestMergeParent` row combines a long beam slug with `held`: the cut and the lookup share one map, and the existing long-slug row pins the cut.
- Legacy `self` run directory appears in `RunIDs` as the name `self`: the old inline reader listed the same directory the same way, so behaviour is preserved, and `self` is no real beam slug.
- `head[:room]` could panic when the numbered tail exceeds the board's slug length: unreachable, and the cut predates this diff.
- No test drives the `RunRecordSlugs` closure in `fixloop.go`: `RunIDs` is covered by `TestRunIDs` and the closure is a one-line delegation to it.
- `fixloop.go`, `land.go` and `cli.go` changes: each is licensed by the shared `RunIDs` helper, the new `HeldSlugs` seam or the help text for the new slug rule; the docs that name the rule (`cli.go`, `doc.go`, the `landDeps` comment) agree with the code.
