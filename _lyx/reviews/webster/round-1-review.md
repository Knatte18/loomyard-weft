---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/fabricengine/scratchlink_integration_test.go:534
    summary: The weft-worktree-absent guard is pinned only on the absent-root branch, not on the adopt branch whose rename it exists to protect, nor on the restore branch
    fix: In TestWireJunctions_ScratchLinkRefusedAfterClassify add a row that arranges a real root .scratch with a file and an absent target, swaps scratchPathTracked through SetScratchPathTrackedForTest to remove RecordsWorktreePath(l, slug) and return false, runs wireWithRecord, and asserts the root is still a real directory holding its file and the target is absent. A second row does the same for a correct link with a removed target and asserts the target is not recreated.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/fabricengine/scratchlink.go:62
    summary: No test pins that withoutRootScratchLink keeps a name unchanged at a subpath anchor, so removing the anchor guard would pass every test
    fix: In scratchlink_integration_test.go add a row on a hubforge.Shape with its Anchor set to backend with a fabric-owned link named .scratch inside the anchor directory, and assert that an applied Unwire and a Remove still treat it as an ordinary anchored junction (removed, counted in JunctionsRemoved or LinksRemoved) while the root .scratch link is untouched by Unwire.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/junction.go:86
    summary: WireJunctionsWith's doc comment still describes junctions and exclude entries only, though the function now also runs the scratch step and logs a left outcome
    fix: In the doc comment of WireJunctionsWith, add that after the junction set it runs the scratch step, which never fails wiring and reports a leave or failure through logger.Warn.
    origin: handler
  - id: F4
    severity: NIT
    class: design
    location: internal/fabricengine/scratchlink.go:124
    summary: When resolving the exclude path itself fails, mutateGitExclude returns an empty path and the report names an empty location after scratch exclude not written at
    fix: In the exclude-failure branch of wireScratchLink, when excludePath is empty, name the warp worktree path in its place, so the report always names a location.
    origin: lens:code-correctness
---

### [LOW] Weft-absent guard untested on adopt and restore

**Location:** internal/fabricengine/scratchlink_integration_test.go:534
**Issue:** `WeftWorktreeGoneDuringTheStep` is the only test of `weftWorktreePresent`, and it reaches the absent-root branch only. The `adoptScratchDirectory` guard protects an `os.Rename`, the riskier case, and the `restoreScratchTarget` guard has no test either. The diff adds both guards, and removing either would pass every test.

### [LOW] Subpath-anchor branch of withoutRootScratchLink unpinned

**Location:** internal/fabricengine/scratchlink.go:62
**Issue:** The shared decision says the skip drops the one name `.scratch` at a root anchor only. The tests pin the root-anchor drop. No test pins the `AnchorRel != "."` branch, so a nested `.scratch` link at a subpath anchor could start being skipped unnoticed.

### [LOW] Stale doc comment on WireJunctionsWith

**Location:** internal/fabricengine/junction.go:86
**Issue:** The doc comment says the function creates junctions and seeds exclude entries. It now also runs the scratch step and logs a left outcome, so a reader judging the symbol's fitness from its comment misses both.

### [NIT] Empty exclude path in the exclude-failure report

**Location:** internal/fabricengine/scratchlink.go:124
**Issue:** `mutateGitExclude` returns `""` when `resolveGitExcludePath` fails, so the report text has an empty location. The error and the way forward are still present, so the report stays actionable.

## Rejected

- Duplicate coverage of the exclude convergence and `path_renamed` between `RealDirectoryAdoptedIntoTheTarget` and `TestReconcile_ScratchLink/RealDirectoryAdoptedOnAHealthyPair`: the two run through different verbs, card 2 lists the assertions, and no covering test is namable under `PATTERN-test-economy`.
- No assertion on the dry run of `Unwire` for the scratch filter: the filter sits before both paths in `Unwire`, and the plan asks only for the applied-run assertions.
- Missing `PATTERN.md` or `pattern/` changes, and a missing `Pattern-Departure:` trailer: no declared PATTERN change exists in the constraints, and the pattern-fit lens found no entry broken.
- Rollback of `Add` meeting a dangling scratch link: the forced `git worktree remove` unlinks it without following it, so nothing is left behind.
