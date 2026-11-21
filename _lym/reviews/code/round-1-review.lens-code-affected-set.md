```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — what the diff's namespace changes force outside the lines it touches. I found three stale operator-facing messages; the rest of the set is covered.

**List, per change**
- **gitrepo push, lease and delete signatures (`ns` parameter):** every non-test caller passes a namespace. They are `handover.go:275`, `recordsgit.go:351`, `archive.go:160` and `archive.go:240`, `pushclassified.go:181`, `pushclassified.go:834` and `pushclassified.go:919`, `takeoverwrite.go:198`, `destroy.go:1628`, `destroy.go:1635` and `destroy.go:1664`, and the seed pushes. The test callers in `gitrepo` are updated. Covered.
- **ls-remote and fetch listings (`--heads` and `--tags` dropped):** `remoteBranchTips`, `remoteTagRefs`, `lsRemoteHeads`, `fetchMovedHeads`, `originBranchHead`, `remoteHeadTip`, `remoteTagsUnder`, `resolveArchiveTip`, `probeWeftLeftover`, `probeLiveLocalWeft`, `resolveWeftBranch`, `targetWeftTip`, `Add`, `readSideOwnerArchives` and `fetchWeftBranchTip` all go through the namespace. The `refs/heads/` literals left in `remove.go`, `remoteteardown.go:479` and `add.go:498` target the code store, so they are fine. `refscanner.go:557` only allow-lists flags. Covered.
- **`UpstreamSHA` and `Upstream` mapping through fetch refspecs:** `parity_test` and `replay_integration_test` cover it. `doc.go` is updated. Covered.
- **`goGit` open path:** `reindexIfPacksChanged` and `readGitDirExcludePatterns` are updated. The `gitrepo/doc.go:284` bullet still says `PlainOpenWithOptions` is "mandatory", yet the code no longer calls it, so the doc contradicts the implementation (finding 3 below). No other non-test `go-git` open exists. Covered otherwise.
- **`CloneOptions.SingleRepo`, `CloneAndWire` and `fabric.go` flag:** the usage string, `fabric.go` help, `fabriccli/doc.go`, `README.md` and `getting-started.md` are updated. Covered.
- **`checkSubpathBeforeReset` and `recordedAnchorOnWeft` (new `ns` argument):** both callers are updated. Covered.
- **PATTERN landing:** the `PATTERN.md` summary lines match the declared text verbatim. The `pattern/PATTERN-code-conventions.md` Branch bullet and the new `pattern/PATTERN-single-repo-hub.md` land. D9's other drops were already gone from the tree, which agrees with the plan's `d9-drops-already-landed` decision. Covered.
- **Commit messages:** no `Pattern-Departure`, `Loosened-Check` or `Root-Cause` trailers appear in the range. The `Loosened-Check` rule applies to the `DeleteRemoteBranch` test, whose old "remote ref does not exist" tripwire row was removed, but the replacement row asserts stricter behavior. Nothing to flag.

**Findings**

1. **MEDIUM, consistency.** `internal/fabricengine/remoteteardown.go:186` and `remoteteardown.go:199`. The way-forward text tells the operator to run `git -C <weftRoot> push origin refs/tags/<archive-tag>`. In a one-repository records store that explicit refspec has no destination, so it bypasses the configured push refspec and writes `refs/tags/archive/...` on the code origin, outside the namespace. The code-store form at line 186 is correct. Fix: for the records side (line 199), build the message with the full pair, `refs/tags/<tag>:<ns.Tag(tag)>`, using `originNamespaceAt(weftRoot)`.
   - A test pinning this text may exist; I did not search for one.

2. **MEDIUM, consistency.** `internal/fabricengine/remoteleftover.go:52`. The leftover-refusal message says `git push origin --delete %s` for the weft branch. In a one-repository hub a short name cannot reach `refs/lym/heads/`, and D4 says teardown deletes by full namespaced name. Fix: format the weft-branch message with `ns.Branch(weftBranch)` read via `originNamespaceAt`. The warp branch message at line 56 stays as it is.

3. **LOW, consistency.** `internal/gitrepo/doc.go:284`. The bullet "`git.PlainOpenWithOptions` with `EnableDotGitCommonDir: true` is mandatory" is stale now that `goGit` resolves the common dir itself. The diff edited line 288 but not this bullet. Fix: reword the bullet to say `goGit` must resolve the common dir.

I did not verify, within the call budget, whether `remoteDefaultBranch` (`cleanupremotewarp.go:326`) is called with the records store. It reads origin's `HEAD`, and D2 says a records-store `HEAD` read answers `main`. Its callers are `addadopt.go:68`, which uses `l.WorktreePath()`, the code store, and `cleanupremotewarp.go:87`, which uses `repoDir`. If `repoDir` there is the code store there is no defect.

**Tool calls (1)**
- One `Bash` call: read `cleanupremotewarp.go` 318-345 and `remoteteardown.go` 180-202, and grepped callers of `remoteDefaultBranch` and `--delete` messages. My context lacked all of these.
