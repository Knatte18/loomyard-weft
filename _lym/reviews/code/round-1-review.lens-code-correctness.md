```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran the cases below from the diff and the manifest.

**Per function or branch: case run, correct or not**
- `state.ReadJSON`, lock directory absent: the lock open returns ENOENT, wrapped by `lock` with `%w`, so `errors.Is(err, fs.ErrNotExist)` holds. The read continues unlocked. Correct. The data directory absent too gives `ReadFile` not-exist, so (zero, false, nil) and nothing is created. Correct.
- `state.ReadJSON`, lock path under a non-directory (ENOTDIR) or permission denied: not not-exist, so it fails with "acquire read lock". Correct, and bounded as the decision record says.
- `readJSONUnlocked`, path is a directory (EISDIR): returns `ErrRead` wrapping the cause with `%w`. Correct. Malformed JSON returns `ErrDecode`. Correct. `UpdateJSON` shares this function, so its read errors now also wrap `ErrRead`. That is harmless.
- `ReadOrigin`, weft `.weft` directory absent: the lock path is joined directly, the lock is skipped, nothing is created, and the result is not found. Correct. `.weft` present: it locks as before. Correct.
- `InWorktreeStaging`, path `<hub>/.fabric-wt-staging-ab/slug`: a component matches the prefix, so true. A normal path gives false. Correct. The check is by component prefix, so any hub ancestor directory with that prefix would also match, which is unlikely.
- `LymJunctionWiredHere` / `fslink.IsLink`: a missing `_lym` returns (false, nil) on both Linux and Windows, so `HubOrchs` skips silently with no warning. A real `_lym` directory returns false, so it is skipped without a warning. A non-not-exist Lstat error gives a warning and a skip. Correct.
- `HubOrchs`, staging worktree: `InWorktreeStaging` is checked before `ResolveWorktree`, so nothing is resolved or read. Correct. An unwired pair is skipped before `ReadOrigin`, so it creates nothing. Correct.
- `Topology.add`, wiring moved ahead of the local marks and `writeLaunchers`: a wiring failure calls `rollbackAdd` with the same arguments as before. A later failure (mark or launchers) now rolls back with junctions already wired. The old order already reached that state when the record commit at step 10c failed, so I found no new defect. The `beforeWire` hook is nil in production and sits after weft creation. Correct.
- `websterengine.LoadState`, scratch directory absent: `ReadJSON` skips the lock and returns not found, so the function returns (nil, nil) and creates nothing. Correct.
- `checkRunStatus` and the `status` verb, lock directory absent: they read without the lock and create nothing. Correct. `pause` and `goto` still create the directory before writing, and their flag comment now says so. Correct.
- `CheckSeed`: the `MkdirAll` is dropped and the read goes through `ReadJSON`. Correct.

**Findings**
1. LOW, consistency, `internal/loomengine/seed.go:124` (in `VerifySeedOwnership`). It still runs `os.MkdirAll(filepath.Dir(statusLockPath))` before its `ReadJSON`. That is a read that creates a directory, which the board entry's "a read never creates a directory, anywhere" forbids, and it is now redundant. The removed `ReadJSONNoMkdir` and the `CheckSeed` sibling both dropped the same call. It runs in a run's own worktree rather than from the hub reader, so the race is not reachable. Fix: delete the `MkdirAll` and the imports it alone needed.

No other defects found in my lens.

**Tool calls**
1. `grep` for `IsLink` in `internal/fslink` and for `EnsureStatusLockDir` users: the manifest lacked `IsLink`'s not-exist behaviour and the remaining `ensureStatusLockDir` callers.
