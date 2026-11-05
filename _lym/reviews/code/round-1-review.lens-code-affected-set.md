```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens review of the `.git`-in-shared-temp guard (commit 3bd03208).

**Per-change set**

- `Main` now records and checks `.git` in the shared temp directories.
  - Set found: the `tmuxkit.go` package doc (updated); `pattern/PATTERN-test-isolation.md` "Private temp directory and leftovers" (updated, two bullets that match `Main`: exit 1, naming, removal of an empty directory, bound); `PATTERN.md` (one-line `PATTERN-test-isolation` entry, which names no `Main` detail); `PATTERN-test-speed.md` (tier1 row says `tmuxkit.Main` checks tmux and LLM use at the end of the package; the new check is not mentioned, but the row is not exhaustive). Covered.
  - Callers: 425 `tmuxkit.Main` call sites, which are unaffected because the signature is unchanged. Covered.
  - Windows and Linux paths: both call `checkNewGitEntries`, and the snapshot is taken before `setEnv`. Covered.
- `sharedTempDirs`, `presentGitEntries`, `checkNewGitEntries` and `gitEntryName` are new unexported symbols. Only `tmuxkit.go` and its test reference them. `internal/hubgeom/hubgeom.go:51` already declares an unexported `gitEntryName = ".git"`. It is in another package and `tmuxkit` is a test kit, so it is not a `PATTERN-named-constants` violation (the pattern concerns repeated literals). The duplicate constant is a NIT at most.
- The new stderr messages: nothing greps for or asserts them outside `tmuxkit_test.go`. Covered.
- The `PATTERN.md` and `pattern/` diff: the rubric lists no declared PATTERN changes, so there is nothing to land. The commit message has no `Pattern-Departure:` trailer, and none is needed.

**Findings**

1. LOW, consistency. `internal/testkit/tmuxkit/tmuxkit.go:17-20` (package doc) and `pattern/PATTERN-test-isolation.md:54-56` state that the check watches the "system temp directory above" the lym root.
   - Summary: when a process inherits a run directory as `TMPDIR`, `resolveRootPath` (`internal/lymtemp/root.go:53-61`) returns the parent's root, so `filepath.Dir(root)` is still the real system temp directory. The docs hold in that case.
   - If `os.TempDir()` is symlinked, `Dir(Dir(dir))` is the unresolved path and the `.git` check still works. No defect from this.
   - The only gap is that the docs do not say the system temp directory is `os.TempDir()` at startup. I count this as a NIT, not a defect.
   - Evidence: `root.go:36`, `root.go:53-61`.
2. NIT, consistency. `tmuxkit.go:109` duplicates the name of `internal/hubgeom/hubgeom.go:51` (`gitEntryName`). It is a separate package and a kit that deliberately imports almost nothing, so no change is needed unless the author wants a shared constant. Not blocking.

No BLOCKING or MEDIUM findings in this lens.

**Tool calls**
1. `grep -rln "tmuxkit.Main|GIT_CEILING_DIRECTORIES" --include=*.md .` plus `grep gitEntryName internal/hubgeom/hubgeom.go`: to find other docs describing `Main`'s checks, and to confirm the duplicate constant. My context had neither.
