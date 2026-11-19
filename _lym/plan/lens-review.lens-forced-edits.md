```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change forces outside the members it names.

**BLOCKING**
1. **Card 3, `PushCoalesced` re-sign:** a caller is left uncovered, so the `fabricengine_test` package stops compiling.
   - `internal/fabricengine/stencilcommit_integration_test.go:58` calls `board.PushCoalesced()` inside `internal/fabricengine_test#TestCommitSeededStencils_ScopedCommitExcludesUnrelatedDirt`, and card 3 does not list it.
   - Card 3's ImpactSummary says "every caller is in fabricengine … or a gitrepo test". That misses this `fabricengine_test` caller.
   - Fix: add the test's glyph under card 3's `**Edit:**`, passing `gitrepo.RefNamespace{}`.
2. **Card 5, `pushCloneSeed` and `probeCodeOriginRoot`:** the destructive guard in `cmd/lym` will fail, and no card lists it.
   - `cmd/lym/destructiveguard_test.go:79-80` bans `.UpdateRemoteBranchLeased(` in every non-test `internal/fabricengine` file except the allowlist (`destructiveGuardAllowlist`, from line 87). The allowlist holds `destroy.go` but not `clone.go` or any new file.
   - `pushCloneSeed` calls `UpdateRemoteBranchLeased` twice, for the seed and for the restore, and card 5 names no file for it.
   - `probeCodeOriginRoot` removes a throwaway clone, which bans `RemoveAll(` too. Only `recordsprobe.go` and `resetsubpathcheck.go` are allowlisted for that.
   - Fix: put the leased pushes and restore in `destroy.go`, or call them through its executors (`updateRemoteBranch` / `deleteRemoteBranch` with `leaseSHA`), and put `probeCodeOriginRoot` in `recordsprobe.go`. Otherwise card 5 lists `cmd/lym#destructiveGuardAllowlist` under `**Edit:**` with a reason per file.

**MEDIUM**
3. **Card 3, `DeleteRemoteBranch`'s new `ls-remote` probe:** it must stay inside `DeleteRemoteBranch`'s own body, or the gitrepo boundary guard breaks.
   - `cmd/lym/gitrepoboundary_test.go:26-62` (`gitrepoPinnedRunBoundMethods`) requires the methods that call `r.run`/`r.runChecked` to equal the pinned list exactly.
   - If the probe is a new helper method, for example a ref-present read, that helper must be added to the list, and card 3 lists neither the constraint nor the guard.
   - Fix: state in card 3's Intent that the probe runs in `DeleteRemoteBranch`'s body, or list `cmd/lym#gitrepoPinnedRunBoundMethods` under `**Edit:**`.

**LOW**
4. **Card 3, `TestDeleteRemoteBranch`'s new namespaced rows:** they cannot see their refs with the existing reader.
   - The shared reader `remoteHeads` (`internal/gitrepo/deleteremotebranch_integration_test.go:28-32`) runs `ls-remote --heads`, which never lists `refs/lym/heads/*`.
   - The helper is not listed. Fix: list `internal/gitrepo_test#remoteHeads` under `**Edit:**`, or say the namespaced rows read with a pattern.
5. **Card 7, `Hub.RecordsBare`:** its field doc becomes false.
   - The field doc ("this hub's own copy of the weft bare remote", `internal/hubforge/hub.go` `Hub` struct) is wrong once `NewOneRepoHub` sets `RecordsBare` to the one code bare.
   - `CloneSecond`'s doc says "from h's bares" and stays true. `internal/hubforge#Hub` is not listed.
6. **Card 8, `primaryWeftBranch`:** a comment the card does not list becomes false.
   - `internal/fabricengine/cleanup.go:31` says "primaryWeftBranch names the primary weft branch, the default branch plus "-weft"". In a one-repository hub it is the configured key, even after a checkout moves the prime.
   - Card 8 lists only the function, not the comment that encloses that line.
7. **Card 5, `ensureBoardWorktree`:** its file header becomes false.
   - `internal/fabricengine/boardweft.go:2` says "The Bolt checkout uses the unsuffixed warp branch". In a one-repository hub the board is `main` whatever the code branch is.
   - Card 5 rewrites only `ensureBoardWorktree`'s parameter and doc comment.
   - The same applies to `internal/fabricengine/clone.go:3-12`, the header describing the weft clone and suffixing, which says nothing of the `git init` store.

I checked every `Edit`/`Uses` glyph across cards 1-9 with `lym quarry resolve`, and none came back unresolved.

Tool calls beyond my context:
- grep of every caller of each re-signed member: the pre-read had no caller list for `PushCoalesced` in `fabricengine` tests, for `cloneHub`, or for `checkSubpathBeforeReset`.
- The members enclosing six caller lines, `cmd/lym/destructiveguard_test.go`, and tests asserting `--heads`/`--tags`: the pre-read had only the guard's file name from the 13cebe3d stat.
- `lym quarry resolve` over the cards' glyphs, first pass: its output was too verbose to read statuses.
- `lym quarry resolve` again filtered to non-found statuses, plus the package clauses of the test files: to confirm the glyphs and the `_test` packages.
- The `ls-remote`, upstream and clone text in the `fabricengine`, `gitrepo` and `fabriccli` doc.go files and in `docs`: to find text the changes falsify.
- `cmd/lym/gitrepoboundary_test.go`, the `archiveTagPushHook` seam convention and the `cleanup.go:25-35` comment: none of these were in the pre-read.
