```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, listing for each change what it forces outside the lines it touches.

**Per-change set**

1. `state.ReadJSON` drops its MkdirAll, and `ReadJSONNoMkdir` is removed.
   - A repo-wide grep finds no `ReadJSONNoMkdir` reference left in code or docs. Covered.
   - The callers are the moved-over sites in the diff plus `UpdateJSON`, which shares `readJSONUnlocked` and now wraps `ErrRead`. The wrap changes only the message text and adds a sentinel, so nothing breaks. Covered.
   - Raw `os` writers beside lock files: the remaining `MkdirAll` sites in battencli and loomcli (cancelcmd.go:287, wire.go:656, resume.go:51) all create their own directories before locking. None relied on a read's mkdir. Covered.
   - Removing `EnsureStatusLockDir` from `statusCmd` leaves the field in use for pause and goto (pause.go:109, goto.go:28). The field comment (spec.go:209) matches the new behavior. Covered.
   - Test and docs updates are `state/doc.go`'s new "Reads create nothing" section, the strict_test header, `TestStatusCmd_AbsentLockDirReadsWithoutCreatingIt`, and the `CheckSeed` and `VerifySeedOwnership` comment updates. Covered.
   - `shuttleengine/attach.go:449` says `ReadJSON` still creates a lock file in an existing directory. That remains true under the out-of-scope clause. Covered.
   - `docs/benchmarks/test-redundancy.md` mentions old test names. It is a measurement report and exempt.

2. `ReadOrigin` builds its lock path without creating it.
   - `originLockPath` and `ensureWeftLockDirAt` keep their other callers: `WriteOrigin` (origin.go:153), originreconcilemoves.go:67, snapshotuncommitted.go:48 and recordsgit.go:48. The creating path is kept. Covered.
   - `ReadOrigin` has no other caller whose behavior needed the mkdir. Covered.

3. `websterengine.LoadState` drops its MkdirAll.
   - The doc comment still says a missing file returns `(nil, nil)`, which holds because a missing lock directory reads unlocked. Covered.

4. `HubOrchs` skips staging and unwired pairs.
   - Its callers are battencli/overview.go:362 and tendercli/deps.go:392. Both inherit the skip, and neither asserts the old behavior. Covered.
   - `orchcli/doc.go` carries the new skip line, and the `HubOrchs` doc comment is updated. Covered.
   - `fabricengine.InWorktreeStaging` and `LymJunctionWiredHere` are new exports. Their only consumer is orchcli, and orchcli names no fabric path. Covered.

5. `Topology.Add` reorders junction wiring.
   - The rollback arguments are unchanged at the moved site. The top-of-file comment in add.go is updated. The `beforeWire` seam is exported only through `export_test.go`.
   - I found no stencil or doc elsewhere that cites the old "(10b)" order. Covered.

**Findings:** none. Every member of the affected set I found is consistent with the diff.

**Tool calls**
1. grep for `EnsureStatusLockDir` and `ensureStatusLockDir`, and for stale "ReadJSON creates / mkdir" comments. My context lacked a repo-wide search for these.
2. grep for `HubOrchs` callers, `MkdirAll` sites in battencli, loomcli, shedverbs and shedengine, and PATTERN mentions of `ReadJSON`. My context lacked the caller list and the sweep for raw writers that relied on a read's mkdir.
3. Read `shuttleengine/attach.go` around line 449, the `LoadState` doc comment, and the `ensureWeftLockDirAt` and `originLockPath` callers. My context lacked them, to check that stale comments and the remaining creating callers still hold.
