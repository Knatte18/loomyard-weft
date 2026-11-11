```yaml
lens: fix-proposal
outcome: reported
```

HubPath comes from the symlink-resolved cwd (`EvalSymlinks`), while the git-listed worktree paths come from `fabricengine.List`.

Task: challenge the F1–F7 proposals the fixer carried out in commit aea1dd58.

**Findings**

1. **MEDIUM** — `renderEveryWorktree` in `internal/heddlecli/verbs.go`, the new `filepath.Dir(path) != hubPath` check from the F1 fix. The comparison is a plain string compare of cleaned paths. If `entry.Path` differs from `c.location.HubPath` only in spelling, every worktree is skipped. The verb still reports ok, and `rendered` and `failed` both come back empty with no warning.
   - Evidence: `lymcwd.gitWorktreeRoot` (`lymcwd.go:152-162`) sets `HubPath` from `filepath.EvalSymlinks(cwd)`, so it is symlink-resolved.
   - On Windows, drive-letter or path case can differ between git's output and the resolved path. `lymcwd` itself handles this in `samePath` with `strings.EqualFold` on Windows.
   - A hub reached through a symlink, or a git that reports a non-resolved path, would produce the same silent skip.
   - The tests ran on Linux with a plain `TempDir`, so they cannot see it.
   - I did not read `fabricengine.List` or run any of this. The case is built from the `lymcwd` code only.
   - Suggested change: compare with the same normalization as `samePath`, or skip only entries that have a different parent *and* are provably outside the hub. At minimum, add a warning when the filter leaves no entry.

2. **LOW** — the F3 test in `WriteRendersEveryLiveWorktree`. The failure branch is driven by an unusable anchor marker, so the failure comes from `NamedWorktree`. The `EventFor` and `Render` error paths of the `err != nil` case stay untested. The review named the Bolt-lock failure as the alternative, and the fixer's choice is sound because a held lock would fail the prime too. The coverage gap is partial.

3. **LOW** — the F4 assertion in the remove step. `strings.Count(..., "lym heddle render") != 1` pins the dedupe. It also passes if the verb-level way forward is missing, as long as the engine's own warning text names the verb once. Because the engine's text omits the worktree name, the "render <worktree>" way forward that `doc.go` promises is not what the caller gets in that case. `doc.go` says each failed worktree comes with the `lym heddle render <worktree>` way forward, which is not literally true for an `Unwritten` failure whose warning already names the verb. This follows the reviewer's own F4 proposal, so it is a proposal weakness rather than a fixer deviation.

F2, F5, F6 and F7: no breakage found. The F6 sync-order assertion fails if the sync is moved before the renders. In the F2 changes, the `OverrideVerbs` sync count of 4 matches the number of override writes.

**Tool calls**
1. `grep` on `gitWorktreeRoot` and `func List` in `lymcwd` and `fabricengine`. I needed to see how `HubPath` is derived against how listed paths are produced. The `lymcwd` half answered; the `fabricengine.List` grep returned nothing, so the list side is unverified.
