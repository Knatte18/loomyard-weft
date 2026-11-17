```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff's changed functions, branches and error paths.

**Case list**

- `removeTreeMakingWritable` (`internal/lymtemp/sweep.go:396`)
  - Path already gone: `Lstat` gives ENOENT, which is filtered, and `os.RemoveAll` returns nil. **Correct.**
  - Dead run directory holding a 0o555 directory with a 0o000 directory below it: the walk chmods both, then `RemoveAll` succeeds. **Correct.**
  - `makeTreeOwnerWritable` fails with EPERM or EACCES. Example: a subdirectory owned by another uid, left by a test run under sudo. The error is returned and `os.RemoveAll` is never tried. **Regression, finding 1.**
- `makeTreeOwnerWritable` (`sweep.go:408`)
  - Root is a symlink: `Lstat` returns no dir, so it returns nil. **Correct.**
  - Directory at 0o000: chmod to 0o700, then `ReadDir`. **Correct.**
  - Directory at 0o300 (no read bit): `perm&0o700 != 0o700`, so it chmods to 0o700. **Correct.**
  - Directory at 0o700 owned by another user, with no access: chmod is skipped and `ReadDir` fails with EACCES. The whole removal aborts even when the directory is empty and its parent is writable, a case `RemoveAll` alone would remove. **Same defect as finding 1.**
  - Inner entry is a symlink to a directory: `DirEntry.IsDir()` is false (the type comes from the link itself), so the link is not followed. **Correct.**
  - Entry vanishes:
    - between `ReadDir` and the recursive `Lstat`: ENOENT, filtered at `sweep.go:429`;
    - between `Lstat` and `Chmod`: the chmod PathError satisfies `fs.ErrNotExist` and is filtered by the caller;
    - before `ReadDir`: ENOENT, filtered by the caller.

    **Correct in all three.**
  - Windows directory with the read-only attribute: reported as 0o555, chmod to 0o755 clears the attribute. A writable directory is reported as 0o777, so it is skipped. **Correct.**
- `removeRunDirectory` with the new default `remove`:
  - A chmod or `ReadDir` error is no `isRemovalSymptom` (the op is chmod or open, and the error is not ErrExist), so it is reported after one attempt.
  - On the not-empty symptom each retry walks the tree again. That is harmless.

  **Correct.**
- `Sweep(false)` never calls `r.remove` (`sweep.go:69`), so a dry run removes and chmods nothing. **Correct.**
- Tests that replace `root.remove` (the retry rows in `sweep_test.go`) bypass the walk, as before. **Correct.**
- `permkit.Chmod` (`internal/testkit/permkit/permkit.go:19`)
  - Missing path: `Stat` fails and the test stops with `Fatalf`. **Correct.**
  - Path removed by the code under test before cleanup: the restore error is dropped, as documented. **Correct.**
  - Called twice on one path: the cleanups run last-in first-out and end at the original bits. **Correct.**
  - Path with setgid or sticky bits: the restore passes only `info.Mode().Perm()` and clears them. **Finding 2.**
  - Windows directory at 0o000: the read-only attribute is set, and the restore to 0o777 clears it. **Correct.**
- Converted call sites:
  - configengine, reedcli and reedengine: they now drop the restore error instead of reporting it, which does not change behavior.
  - gatecli and impactset: the restore is the same as before.
  - fabricengine destructivegaps and remove_leftoverdir: the mid-test chmod to 0o755, the `RemoveAll`, and the cleanup chmod failing silently behave as before.
  - Every other site now restores the original bits instead of 0o755. Those bits are the 0o700 or 0o755 of a `t.TempDir` path, which still leaves the tree removable.

  **Correct.**
- `TestSweep` row `lockedTree` (`sweep_test.go:91-106`)
  - `permkit.Chmod(unreadable)` then `permkit.Chmod(readOnly)`: `Stat` of `readOnly` works. The cleanups run last-in first-out, so the parent's bits come back first.
  - `Chtimes` after the mkdirs resets the run directory's age.
  - The `t.TempDir` cleanup is registered earlier, so it runs after the restores.

  **Correct.**

**Findings**

1. **LOW**, class `design`. In `internal/lymtemp/sweep.go:397` with `:416-424`, an error from the owner-writable pre-pass aborts the removal outright. Any chmod or `ReadDir` failure other than not-exist (EPERM on a foreign-owned subdirectory, or EACCES on a 0o700 directory owned by another user) returns before `os.RemoveAll` runs. Before this diff, `os.RemoveAll` alone removed such a tree whenever the entry's contents were removable, for example an empty foreign-owned directory under a writable parent. Now the sweep reports a failure and leaves the tree in place.
   - Evidence: `sweep.go:397-400` (`return err` before `RemoveAll`), `sweep.go:417-419` and `sweep.go:421-424`.
   - Fix: make the pass best-effort. Ignore or collect its errors and let `os.RemoveAll` decide, reporting its error.
2. **NIT**, class `design`. `internal/testkit/permkit/permkit.go:28`: the restore passes `info.Mode().Perm()` and clears any setgid or sticky bit the path had. A directory that inherited setgid from a g+s parent comes back without it. The doc at `permkit.go:16` promises "the bits path had before". This does not affect removability.
   - Evidence: `permkit.go:21-28`.
   - Fix: restore `info.Mode() & (fs.ModePerm|fs.ModeSetuid|fs.ModeSetgid|fs.ModeSticky)`, or say "permission bits" in the doc.

Tool calls: none. The inherited context held every definition and call site I needed.
