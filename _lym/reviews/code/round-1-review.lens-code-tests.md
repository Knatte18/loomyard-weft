```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds, the test that pins it, or none.

**Per-behaviour list**

1. `presentGitEntries` records any `.git` entry, whatever its type, in a before-snapshot. Pinned by `TestCheckNewGitEntries/EntryPresentBeforeTheRunKeepsTheCodeAndTheEntry` (`tmuxkit_test.go:126`). A broken snapshot would flag the preexisting entry and fail that row.
2. `checkNewGitEntries`, no new entry: code unchanged, no output. Pinned by `NoEntryKeepsTheCode`.
3. `checkNewGitEntries`, new empty directory: exit 1, named path, directory removed. Pinned by `NewEmptyDirectoryFailsAndIsRemoved`. It checks the code, both messages, the path and `wantKept=false`.
4. `checkNewGitEntries`, new non-empty directory: exit 1, kept, "remove it by hand". Pinned by `NewNonEmptyDirectoryFailsAndIsKept`. This covers the `os.Remove` failure branch.
5. `checkNewGitEntries`, new gitfile (non-directory): exit 1, kept. Pinned by `NewGitfileFailsAndIsKept`. This covers the `!info.IsDir()` branch.
6. `checkNewGitEntries`, a new entry in each watched directory, and the code staying 1 across several entries: none. Every row seeds only the second directory (`dir`). The first directory (`t.TempDir()`, never seeded) is never tested with an entry. A bug that scanned only `dirs[1]` or only `dirs[0]` would pass if it kept the second. Only the "scan dirs[0] only" bug would be caught. There is no row with a new `.git` in the first directory.
7. `checkNewGitEntries` with an incoming `code` of 1: the code is preserved. None. Every row passes 0. Preserving an incoming nonzero code is trivial, so this is a low-value gap.
8. `checkNewGitEntries` with a symlink `.git`: none. The `Lstat` branch is not exercised for a link, and a dangling symlink is not covered. The Lstat-based `IsDir` makes a link non-removable. A broken Lstat-to-Stat swap would remove or follow it, and no row would fail.
9. `sharedTempDirs` returns the root and the system temp directory above it. Pinned by `TestSharedTempDirs_AreTheRootAndTheSystemTempDirectoryAboveIt`.
10. `Main` wiring (the snapshot taken before `m.Run`, the check run after it, the exit code propagated, on both the Windows and Linux paths): none. `Main` takes a `*testing.M` and is covered only indirectly, since this package's `TestMain` runs through it. No test fails if the snapshot or the call is dropped from `Main`, or if the call moves before `m.Run` or after `afterRun`. This is the single behaviour the change exists for, and it is covered only by the helpers.

**Findings**

- **MEDIUM, design** — `internal/testkit/tmuxkit/tmuxkit.go:87-105` (evidence also `tmuxkit_test.go:112-199`). Behaviour 10 has no test. The helper tests would still pass if `Main` dropped the snapshot or the check, or took the snapshot after `m.Run`. `Main` takes `*testing.M`, so a direct test is hard. The usual route is a package-internal child process driven by an environment variable, as `TestMain` allows for child bodies. A cheaper route is to extract the Linux and Windows duplication into one function taking a run func, and test that function with a fake run that creates a `.git`. The current duplicated call on both paths also makes the wiring easy to break on one OS only.
- **LOW, design** — `tmuxkit_test.go:124-160`. Behaviour 6 has no test. All rows seed only `dirs[1]`, so a regression that watches only one of the two shared directories is not caught. Fix: add a row, or a seed target field, that places the new `.git` in the first directory.
- **LOW, design** — `tmuxkit_test.go:150-160`. Behaviour 8 has no test. A `.git` symlink is the only case that exercises `Lstat` against `Stat` in `checkNewGitEntries`. Fix: add a row seeding `os.Symlink("/elsewhere", path)`. It expects exit 1 and kept, with "remove it by hand". A symlink may be unavailable on Windows, so gate it by OS the way `PATTERN-test-speed` prescribes.
- No test-economy finding. The five rows are one table, each covering a distinct branch, and the sharedTempDirs test pins different behaviour. The tests write only under `t.TempDir()`, and all call `tiergate.Require(t, 0)` and `t.Parallel()`.

**Tool calls:** none made. The manifest already held everything needed.
