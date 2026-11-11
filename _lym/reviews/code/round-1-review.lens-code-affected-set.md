```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. I checked what the diff's removals and renames force outside the lines it touches.

**Per change, the set found, covered or not**
- **Launcher writers and teardown removed** (`writeLaunchers`, `removeLaunchers`, `restoreLaunchers`, `removeMenuLauncher`, `pruneEmptyAncestors`, `LauncherDir`, `menuLauncherPath`): no Go caller is left. The only hits are the one doc comment and one test comment in finding 1. Covered.
- **`PairParts.Launchers`, `pairState.launchers`, `RemoveStepLaunchers`, `ReconcileActionLaunchersRestored`, `hubforge.PairLauncherDir`:** the `battencli` consumers were updated in the diff. A repo-wide grep finds no other reader or assertion outside `docs/benchmarks` and `docs/research`, which are measurement records and exempt. Covered.
- **`refuseWhenNothingRemains` signature (drops `localMode`):** its one caller was updated. Covered.
- **`launchersDir(l)` became `launchersDir(hubPath)`:** the `portals.go` call sites were updated. `destroy.go:576` and `destroy_test.go:629` still use `launchersDirName` correctly. Covered.
- **`readRetiredPortalDir` renamed to `readRetiredDir`:** no stale reference remains. Covered.
- **`lym cleanup` `launchers` segment:** the segment code, `cli.go` Short and Long help, `cleanupcli/doc.go`, `docs/overview.md` (lines 148-149 and 782), `SANDBOX-FABRIC-SUITE.md` F17 and `pattern/PATTERN-cwd-resolution.md:50` all describe it. Covered.
- **`fabriccli` help text:** the `remove`, `checkout`, `reconcile`, `prune` and `anchor` help text is updated. `fabric.go:272` still describes the reconcile sweep of the old workspace file, which still exists in `portals.go`. Covered.
- **Pattern files:** `PATTERN-fabric-boundary` and `PATTERN-cwd-resolution` are edited and consistent with the code. The directive declares no PATTERN changes, so there is nothing to check there. No commit carries a `Pattern-Departure:` trailer, and none is needed. Covered.
- **Reserved name `_launchers`:** kept in `HubReservedNames`, `launchersDirName` and the `junctionnames_test.go` rows. This conflicts with the board entry; see finding 2.

**Findings**

1. **LOW, consistency.** `internal/fabricengine/doc.go:1564` cites `refuseUncontainedPath` as the containment check, but the diff deletes that function from `ancestors.go`.
   - The surviving function is `containmentFailure`; `containmentPath` is also named in that sentence.
   - `internal/fabricengine/destroy_test.go:148` has the same dangling comment: "refuseUncontainedPath resolves through filepath.Rel".
   - Evidence: grep hits at those two lines, and the `ancestors.go` hunk that removes the function.

2. **MEDIUM, scope.** The board entry (`decision-record.md`, `## Goal`) asks to take "the reserved _launchers name out of fabricengine". The diff keeps it reserved.
   - It keeps `launchersDirName` and returns it from `HubReservedNames` (`junctionnames.go:237`).
   - It keeps `_launchers` as a geometry root in `destroy.go:583`.
   - It states "stays reserved" in `doc.go:1672`, `PATTERN-cwd-resolution.md:50` and `docs/overview.md:148`.
   - The pinning rows are in `junctionnames_test.go:78,206,227,259`.
   - The first commit message says "_launchers stays a reserved hub name", but there is no `Pattern-Departure:` trailer, no `Added after Discussion` entry and no recorded parent correction.
   - A kept reservation can be defended, because a hub may still hold the directory. As it stands, though, the deviation from the amended entry is unrecorded. The fixer should leave the code and defer it to the parent via `lym loom decision add`.
   - Evidence: the diff hunks above, and `decision-record.md` lines 3-7 and 13-17, which hold no amendment on this point.

**Tool calls (4 of 6)**
1. Grep for the removed or renamed symbols repo-wide, to find leftover callers. My context had only the diff, not the whole tree.
2. Grep for "launcher" under `docs`, `pattern` and a few source paths. Its 60-line cap was filled by `docs/research`, so it showed little beyond that noise.
3. Grep of test files for launcher references, to find stale test assertions. The earlier file list was truncated.
4. Re-run of the "launcher" grep excluding `docs/research` and test files, to see the real remainder.
