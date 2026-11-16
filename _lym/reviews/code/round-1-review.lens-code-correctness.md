```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff's changed functions and branches for correctness through the code-correctness lens.

Each function or branch, the case run, and the result:
- `testMapLocation`, `resolveHub` error: returns `nil, "", "gate testmap: <err>; way forward: re-run…"`. `runTestMap` checks `refusal != ""` and stops (`testmap.go:243-246`). Correct.
- `testMapLocation`, cwd outside every hub (`inHub=false`, `err=nil`): returns the refusal "…way forward: run lym gate testmap from a worktree of a hub". Correct, and it matches the updated `OutsideEveryHub` test row.
- `testMapLocation`, an in-hub task worktree: `coverageMapsDir` gives `testcover.Dir(AnchorPath())`, which comes back as the second return value. `runTestMap` puts it in `run.mapsDir` (`testmap.go:251`). Correct.
- `testMapLocation`, `coverageMapsDir` error: refuses with "gate testmap: hubgeom: cannot tell the prime…; way forward: re-run the same command". Nothing has been acquired yet. Correct.
- `coverageMapsDir`, prime (`.git` is a directory, `hubgeom.go:74`): returns `testcover.Dir(BoltDir(HubPath))`, the shared maps, as before. Correct.
- `coverageMapsDir`, task worktree (`.git` is a gitdir file): returns `<WorktreePath>/<AnchorRel>/.lym/testcover`.
  - With `AnchorRel="."` this is `<worktree>/.lym/testcover`, which matches the new test's `testcover.Dir(pair)`.
  - With a subdirectory anchor it sits beside the anchor's `_lym`, as PATTERN-lym-files' "siblings under `AnchorPath()`" says.
  - Correct.
- `coverageMapsDir`, a pair's records sibling as cwd: `resolveHub` first maps it through `fabricengine.CodeLocation` (`run.go:668`), so the anchor and the prime check are the code worktree's, not the records sibling's. Correct.
- `coverageMapsDir`, `.git` missing or unreadable: `IsPrime` returns its error, which is passed through unwrapped. Both callers prefix it with their verb. Nothing is swallowed. Correct.
- Build lock under the new `mapsDir`:
  - `testMapBuildLockPath` passes the map path to `lock.TrackedFileLockPath`. No ancestor is named `_lym`, so the lock lands at map path + `.build.lock`, beside the map in the worktree's own `.lym`.
  - Two worktrees therefore never share a lock, and two calls from one worktree do share one.
  - `lockPackage`'s `MkdirAll` creates the directory chain on the first call.
  - Correct.
- Dirty-file check under the new `mapsDir`: map and lock files are `.json` and `.lock`, and `dirtyGoFile` looks only at Go files in package directories. The anchor's `.lym` is also ignored, through `.gitignore` here and fabric's info/exclude seeding of the `.lym` junction elsewhere (`junction.go:183`). Writing maps cannot make a package dirty. Correct.
- HEAD-moved check: `readHead` reads `r.fabric.HeadSHA()` for the call's own worktree, and writing maps creates no commit. Correct.
- `runRedundant`, `coverageMapsDir` error: returns "gate redundant: <err>; way forward: re-run the same command" after the hub and module checks. Correct.
- `runRedundant`, task worktree with no map built yet: `ReadMap` returns not found, and the rebuild way forward tells the operator to run `lym gate testmap <pkg>` from this worktree. That is the same directory `testmap` writes from there. Correct.
- `runRedundant`, prime: reads the shared maps as before. Correct.
- `PrimeWorktreePath` removal: grep finds no Go or doc caller left. Correct.
- `testcover.Dir` parameter rename (`boardDir` to `root`): same body, so behaviour is unchanged. Correct.
- `testMapRun` construction: `mapsDir` comes from the argument and the empty slices are unchanged. Correct.

Findings: none. Every branch I ran gives the right result and reports its errors without swallowing them.

Tool calls:
- grep of `DotLymDirName` in `fabricengine/junction.go` and of `isPrimeWorktree` in `hubgeom.go`. My context did not show how a task worktree's `.lym` is excluded from git, or how `IsPrime` fails.
