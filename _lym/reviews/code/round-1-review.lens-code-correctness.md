```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff's changed functions.

**Per-function list**

- `SweepRetiredLaunchers`, no `_launchers`: `Lstat` returns `ErrNotExist`, so the sweep comes back empty with a nil error. Correct.
- `SweepRetiredLaunchers`, `_launchers` is a symlink or a file: `Lstat` does not follow it and `IsDir` is false, so the entry goes to `Kept` as "not a directory" and nothing is touched. Correct.
- `SweepRetiredLaunchers`, unresolvable hub path: `OpenRoot` fails and the error is returned wrapped. Correct.
- `planRetiredLaunchersSweep`, empty subdirectory: it empties, so it becomes a candidate. Correct.
- `planRetiredLaunchersSweep`, symlink child: `Lstat` shows it is neither a directory nor a regular file, so it is kept and its parent chain stays. Correct.
- `planRetiredLaunchersSweep`, foreign file next to fabric scripts: the scripts are listed for removal, the foreign file is kept, and the ancestors are not candidates. Correct.
- `planRetiredLaunchersSweep`, `*.code-workspace` regular file: matched by suffix and removed. Correct.
- `planRetiredLaunchersSweep`, a directory named like a script: it recurses as a directory, so the name check does not apply. Correct.
- `removeRetiredLauncherEntry`, target is the container itself: the container is swapped for `hubPath`, so the gate's containment check passes. Ownership checks the base name of `_launchers`, so it passes. The removal is non-recursive. Correct.
- `removeRetiredLauncherEntry`, a directory that gained an entry after the listing: the OS refuses the non-recursive remove, the error is returned and the sweep stops with the partial `Removed` list. Correct and fail-safe.
- `SweepRetiredLaunchers` dry run: no gate and no recorder call, and the same listing as an apply run. Correct.
- Cleanup `launchers` segment: it returns the map together with the error, matching the other segments. The prime guard is not in the diff, so I could not check it. Looks correct.
- `Add` / `rollbackAdd` / `EnsureAnchor` / `wireAtTarget`: removing the launcher calls does not change the ordering of the remaining steps or the rollback. `EnsureAnchor` now returns the `WireJunctionsWith` error directly. Correct.
- `pairHasLocalPart`: it no longer calls `NamedWorktree`, so an unreadable anchor no longer errors there. The local-part decision now uses only the worktree paths and branches. Correct.
- `inspectPair` / `refuseWhenNothingRemains`: with the launchers condition gone, a pair with no worktree and no branch falls through to the origin ask, then the stray-path error, then `ErrPairNotFound`. Correct.
- `PairParts.Any` / `removePair` steps: no launcher term is left. `RemoveStepLaunchers` was removed, and I found no remaining user in the diff. Correct.
- `readRetiredDir`, `launchersDir(l.HubPath)` in `portals.go`: the signature change is applied at every call site shown. Correct.
- `ancestors.go`: `refuseUncontainedPath` and `pruneEmptyAncestors` are removed. I found no caller in the diff, and the compile gate decides the rest. Correct.

**Findings**

1. NIT, `design`, `internal/fabricengine/retiredlaunchers.go:34-43`.
   - `hubRel` is a general `os.Root` helper but now sits in `retiredlaunchers.go`.
   - It has callers in `hubworkspace.go:220`, `hubworkspace_write.go:30`, `teardownlock.go:56` and `portals.go:105`.
   - Its home file is unrelated to its use, which is a file-organisation issue, not a defect.

No BLOCKING or MEDIUM correctness defect found.

**Tool calls**
1. `grep` for `hubRel(` and `hubRelativeTarget(` callers: the manifest lacked the callers of the helper that the diff moved into the new file.
