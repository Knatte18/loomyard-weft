```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each card's change, I listed what reads it and which card covers it.

**Per change: the set found, and the cards that cover it**

1. **Card 3 adds a namespace parameter to the gitrepo push and delete operations** (`Push`, `PushBranch`, `PushCommit`, `PushRebaseFree`, `PushCoalesced`, `DeleteRemoteBranch`, `DeleteRemoteBranchLeased`, `UpdateRemoteBranchLeased`, `PushWithRetry`, `PushWithRetryReasons`, `retryablePushRefusal`, `namesRef`, `resolvePushDestination`, `pushPlainWithRetry`).
   - Production callers: `pushJob.push`, `pushJob.repushOverOriginContained`, `retryWeftPush`, `retryBoltPush` (`pushclassified.go:172,265,797,878`), `Fabric.PushWeft` (`recordsgit.go:347`), `takeoverBranch.land` (`takeoverwrite.go:194`), `archiveWeftTip` and `pushArchiveTag` (`archive.go:156,232`), `pushOwedCodeTip` (`handover.go:275`), `deleteRemoteBranch` and `updateRemoteBranch` (`destroy.go:1623,1630,1659`).
   - Test callers: every enclosing test function the manifest's mapping produced, including `TestUpstreamSHA` (`replay_integration_test.go:38`), `TestRetryablePushRefusal_…` (`pushretry_test.go:58`) and `TestCommitSeededStencils_…`.
   - `statuscommit`, `loomcli` and `battencli` call injected `Push func() error` fields, not gitrepo, so they are not in the set.
   - Covered in full by card 3.
   - The earlier hits in `pushanchored_integration_test.go` and `teardownlock_integration_test.go` were false grep matches; neither file calls a gitrepo push operation.
2. **Card 3 deletes `remoteRefAbsentTrigger`.** Only `DeleteRemoteBranch` reads it; no test does. Covered.
3. **Card 3 lets `UpdateRemoteBranchLeased` take an empty expected SHA.**
   - The existing rows in `push_test.go:407-413` pass only `"not-a-sha"`, so they hold.
   - `keyvalidation_test.go` does not call it.
   - Fabric callers always pass a non-empty lease: `moveRemoteTaskBranch` returns first on an empty `tip` (`destroy.go:2438`), and `pushOwedCodeTip` moves only when origin holds the branch (`handover.go:259`).
   - Covered.
4. **Card 3 makes the plain delete probe first.** Readers are `deleteRemoteBranch` (`destroy.go:1630`), `TestDeleteRemoteBranch`, `TestPush_AFailedRecord…` and `TestRemoteRefsLock_*` (`remoterefslock_integration_test.go:292`). The pinned-run list already holds `DeleteRemoteBranch` (`cmd/lym/gitrepoboundary_test.go:46`). Covered by card 3.
5. **Card 3 changes `recordPush` to write `branch.&lt;b&gt;.merge` through the namespace.** Readers are `Upstream`/`UpstreamSHA` (card 1) and `HasUnpushed`/`HasUnpulled` through `@{u}` (git-native). Covered by cards 1 and 3.
6. **Card 1 changes how `Upstream` and `UpstreamSHA` resolve the tracking ref.** Callers are in fabric (for example `goneEntryEnded`, `pushclassified.go:660`); the tests are `TestUpstreamSHA` and `TestParity`. Covered.
7. **Card 4 changes the ref names the remote listings and fetches send and parse.**
   - `remoteTagRefs`' pattern becomes relative to the tags prefix. Its only caller is `cleanupRemoteWarp` (`cleanupremotewarp.go:100`), and no test calls it directly. Covered.
   - `remoteHeadTip`'s callers: `targetWeftTip`, `resolveWeftBranch`, `RecordsBranchFile`, `readSideOwnerArchives`, `deleteTaskBranchOnOrigin`, `discardTaskBranchOnOrigin`. The last two run in the code store and read the identity namespace, so they need no edit.
   - Other tests matching on `--heads`/`--tags` (`teardownlock_integration_test.go`, `deleteremotebranch_integration_test.go`) read the bare themselves. `refscanner.go:557` filters commands agents run, not fabric's own argv.
   - Nothing outside the card is forced.
7. **Card 5 adds a namespace parameter to `checkSubpathBeforeReset`, `recordedAnchorOnWeft` and `fetchWeftBranchTip`, and changes `ensureBoardWorktree`'s parameter.** The callers are `CloneHub`, `checkSubpathBeforeReset`, `recordedAnchorOnWeft` and `probeMainWeftHasPath`, all listed. No test calls them directly. Covered.
8. **Card 5 runs an `ls-remote` check before every one-URL clone.**
   - `TestCloneHub_UnreachableWeftIsHardError` asserts the `probe weft &lt;url&gt;:` prefix; card 5 keeps it through `wrapProbeError`.
   - `TestCloneHub_UnboundWeftNamesTwoArgForm`, `warpbinding_test.go:103` and the message at `codebinding.go:150` still hold, since an origin with no board takes today's path.
   - Covered.
9. **Card 6 adds a parameter to `runCloneWithReset` and changes the usage string.** The only caller is `Command`. The usage text is at `fabriccli/clone.go:225`, inside `runCloneWithReset`. `cli_refusals_test.go:55,61` assert only the substring `"usage: lym fabric clone"`, which survives. Covered by card 6.
10. **Card 6 changes what `CloneAndWire` does on a failure.** Its doc comment says "On error, the clone is left intact"; card 6 edits `CloneAndWire`. The other callers are `CloneAndWireFromLocalBares` and hubforge's `cloneHub`, and their behaviour is unchanged for two-URL clones. Covered.
11. **Card 7 adds a parameter to `cloneHub`.** Its callers are `NewHub`, `CloneSecond` and `buildTemplateIn`, all listed. The `Hub` field docs are listed too. Covered.
12. **Card 8 changes where `primaryWeftBranch` reads the prime's records branch, and adds a refusal.** Callers: `cleanup.go:134` and `destroy.go:827,846`, with no signature change. The two-repository refusal text is unchanged. The tests are the two `cleanup_primary` tests, and the `cleanup.go` header sentence is named. Covered.
13. **Doc text the changes touch:**
    - gitrepo `doc.go:185` (`namesRef`): card 3.
    - fabric `doc.go:117,836,1523` (the `ls-remote` descriptions) stay true.
    - The `CloneHub` doc: card 5.
    - hubforge `doc.go`: card 7.
    - `docs/getting-started.md` and `README.md`: card 10.
    - No other doc under `docs/`, `plugins/` or `contracts/` describes the one-URL clone form.

**Findings:** none. Every set my procedure found is fully covered by the listed cards.

Tool calls:
- Grepped `_test.go` files for direct calls of the helpers cards 3–8 change; the manifest's enclosing-function map covered only gitrepo's push operations.
- Grepped for tests matching `ls-remote` flags and read `refscanner.go:545-570` and the push lines of `pushanchored`/`teardownlock`, to settle whether card 4's argv change and the earlier grep hits force edits.
- Grepped for readers of the clone usage string and clone syntax, since card 6 changes the usage string and the manifest held no reader of it.
- Read the clone-syntax messages in `codebinding.go` and `warpbinding_test.go`, and grepped the docs for the one-URL form, to settle whether card 5's one-URL semantics falsify them.
