---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-test-speed.md:221
    summary: PATTERN-test-speed background still says lym gate testmap builds from the hub's prime worktree and plans at the prime's HEAD, which is false for a non-prime call after the diff
    fix: In pattern/PATTERN-test-speed.md, reword line 221 from "which rebuilds the per-package coverage maps from the hub's prime worktree" to say it rebuilds them at the HEAD of the worktree it runs in, the hub's shared maps from the prime and a worktree's own maps under its .lym elsewhere; and reword line 290's "plans every map at the prime's HEAD" to "plans every map at the HEAD of the worktree it runs in". If the repository requires a declared PATTERN change before a pattern/ background is edited, leave the file and defer the edit to the run's parent.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go:641
    summary: The only non-prime test uses the root anchor, so AnchorPath equals WorktreePath and nothing pins that a worktree's maps lie under its anchor rather than its root
    fix: In TestGateTestMapInATaskWorktreeKeepsItsOwnMaps, build the hub with a subdirectory anchor, as the hubforge.Shape with Anchor set to backend at test_integration_test.go:2400 does, seed the module under that anchor, and assert the map lies under testcover.Dir(filepath.Join(pair, "backend")) with the worktree's HEAD as its commit; update the comment at line 667 to match.
  - id: F3
    severity: LOW
    class: scope
    origin: handler
    location: internal/gatecli/testmap.go:95
    summary: The diff drops the Long sentence about the closing audit round, which the board entry does not license and which is still live behaviour, while Long still promises mismatch counts
    fix: In testMapCmd's Long in internal/gatecli/testmap.go, restore a sentence after the line ending "a test the spawn scan marks is never run." saying that once nothing is left queued it re-measures the oldest passing entries if a gate slot is free at once, counting each difference as an audit mismatch. Keep Long within cmd/lym/clitree_test.go's maxLongBytes by shortening the new lines 82 to 85, for example folding "It runs in any worktree of the hub." into the next sentence.
---

### [MEDIUM] PATTERN-test-speed background still describes prime-only testmap

**Location:** `pattern/PATTERN-test-speed.md:221`, and `:290`

**Issue:** Line 221 says `lym gate testmap` "rebuilds the per-package coverage maps from the hub's prime worktree".
Line 290 says a call naming no package "plans every map at the prime's HEAD".
After the diff a call from any other worktree builds at its own HEAD into its own `.lym/testcover` (`internal/gatecli/testmap.go:301-313`).
The diff updated `docs/overview.md` and the gatecli package doc but left this rule document's background false.
Confirmed by reading both lines.

### [LOW] Non-prime test cannot tell the anchor from the worktree root

**Location:** `internal/gatecli/testmap_integration_test.go:641`

**Issue:** The hub is built with `Anchor: "."`, so `location.AnchorPath()` and `location.WorktreePath()` coincide.
A change of `coverageMapsDir` (`testmap.go:313`) to the worktree root would pass every test,
yet the package doc and PATTERN-lym-files place a worktree's `.lym` under its anchor.
The anchor choice is a branch of the added behaviour with no row that pins it.

### [LOW] testmap Long drops the audit round

**Location:** `internal/gatecli/testmap.go:95`

**Issue:** Commit `1d991f18` removed "Once nothing is left queued, it re-measures the oldest passing entries if a gate slot is free at once, counting each difference as an audit mismatch." to keep Long under the 2000-byte cap.
The audit is live (`testMapAuditTests`, `run.audit` at `testmap.go:493`), and Long still says the call prints "the plan's and mismatch counts".
The help is not false, but it no longer tells the operator a call may spend extra time re-measuring passing entries, and the board entry asks for no change to the audit.
The package doc (`internal/gatecli/doc.go:1106`) still carries it, so the omission is in the help alone.

## Rejected

- code-tests' LOW on `redundant.go:99`, the rebuild way forward unpinned by exact text: the no-map error path is covered (`redundant_integration_test.go`, row `APackageWithNoMap`), the suite's convention for these refusal rows is substring matching, and PATTERN-refusal-way-forward binds webster, shed and loom refusals, not gate verbs; a text-only change on a covered path is no coverage gap.

## Focus verdicts

- Audit sentence in Long: not licensed by the entry and not false; filed as F3.
- `coverageMapsDir`: a task worktree resolves to its anchor's `.lym/testcover`; a records-sibling or nested cwd goes through `resolveHub`'s `CodeLocation` first; a prime always has the board directory, since `resolveHub` refuses a hub without it.
  `.lym` is ignored through `.gitignore` here and fabric's info/exclude seeding elsewhere, and `dirtyGoFile` looks at Go files alone, so the maps never trip the dirty-file skip or the HEAD-moved check. Not a defect.
- The prime keeps the shared maps, consistently across `testmap`, `redundant`, the gatecli doc and the overview; only the PATTERN background lags (F1).
- `redundancy.go:282` reading the shared maps: the entry names `lym gate redundant` alone, and the gatecli doc states the report step's behaviour accurately. Not a defect.
- Build lock: `testMapBuildLockPath` derives from `mapsDir` and `lock.TrackedFileLockPath` places it beside the map, so it is per worktree; no exit record remains in the code. Not a defect.
- `hubgeom.PrimeWorktreePath`: no caller or doc reference left.
- Tests: the new test covers the non-prime path, shared-map isolation and `redundant` reading the same worktree's map; `OutsideEveryHub` covers the refusal; no duplication with `TestGateTestMap`, which runs from the prime. Gap filed as F2.
- Doc and comment claims in `docs/overview.md`, gatecli `doc.go`, `testcover/doc.go` and `lymdirs/dirs.go` match the code.
