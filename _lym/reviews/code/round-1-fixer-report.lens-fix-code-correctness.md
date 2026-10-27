```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F5 fix changes through the code-correctness lens.

Findings:

1. **MEDIUM**: the exclude assertion in `TestReadOrigin_CreatesNoWeftDirectoryAndSeedsNoExclude` (`internal/fabricengine/add_hubreader_integration_test.go`) is vacuous.
   - The test builds the pair with `AddPairWith`, so Add has already seeded the weft worktree's `.weft` exclude entry.
   - It then removes only the `.weft` directory and compares `info/exclude` before and after `ReadOrigin`.
   - A regression that makes `ReadOrigin` seed the exclude again would rewrite an entry that is already there, so the file would be unchanged and the test would pass.
   - The `.weft` existence assertion in the same test does fail with the defect in place.
   - The same weakness applies to the exclude check in the `BeforeWire` hook. There, `.weft` already exists before the read, so a `ReadOrigin` that creates it or seeds the exclude changes nothing visible.
   - I have not read the seeding code, so whether it rewrites an existing entry is unconfirmed.
   - To make the check real, strip the exclude entry along with removing `.weft`, then assert it is still absent after the read.

2. **LOW**: the `level=WARN` assertion in `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs` (`internal/orchcli/beam_integration_test.go`) only guards a narrow regression.
   - It fails only if the unwired pair is read and the read errors.
   - A silent re-listing of the unwired pair is already caught by the `want` list comparison, so the warning check adds little.
   - It does no harm, and the pair-skip behaviour is pinned.

Checked and correct:

- **F5** (`internal/loomengine/seed.go`): `ReadJSON` treats a missing lock directory as `fs.ErrNotExist` through the `%w` chain and reads unlocked. Dropping `MkdirAll` and the `path/filepath` import is sound, and `os` is still used.
- **F1** (`internal/websterengine/state_test.go`): the scratch-directory assertion would fail if `LoadState` called `MkdirAll`. It does not interfere with the later `SaveState` in the legacy row.
- **F3** (`internal/state/strict_test.go`): with a directory at the lock path, the lock open fails with a non-not-exist error. A regression that skips the lock on any lock error would return the record and fail the `wantErrLock` row.
- **F4** (`internal/fabricengine/destroy_test.go`): every `TestInWorktreeStaging` row evaluates correctly against `InWorktreeStaging`. Dropping `t.Parallel` for the logger capture is consistent with the existing convention.

Tool calls made: none. My context already held every changed file.
