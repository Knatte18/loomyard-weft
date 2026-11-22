---
dispositions:
  - id: G1
    disposition: fixed-differently
    reason: "The URL comparison is dropped as proposed, and a code store's namespace now follows the layout of its prime's records sibling alone. The namespace it returns, and the ref the new test row asserts, is the code store's own refs/lym/code-tags/ and not the records store's refs/lym/tags/ this fix key names: G4's fix key and the orch brief move the code store's tags to their own prefix, and returning the records namespace would restore G4's defect (internal/fabricengine/archive.go:232)."
  - id: G2
    disposition: fixed
  - id: G3
    disposition: fixed-differently
    reason: "The re-run pushes the local archive tags origin lacks before any archive or delete step, as proposed, with one narrowing: the tag of the tip the job itself carries is left to the job's own archive step. Pushing it in the new step too would record the tag push twice, and would put a discard's local tip on origin where the discard archives only the tip that contains it, which TestDiscardTaskBranch pins (internal/fabricengine/taskbranchdiscard_integration_test.go:74 and :114). A job's own tip is still pushed before any delete, by the step that already did so."
  - id: G4
    disposition: fixed-differently
    reason: "The constant and the namespace change are as proposed. The new test sits in internal/fabricengine/ownerarchive_integration_test.go, not in remoteteardown_namespace_integration_test.go: that file is in the internal package, which cannot import hubforge (internal/fabricengine/export_test.go:186), and the orch brief asks for the test over hubforge's one-repository builder. The internal file's existing row now expects the code store's own prefix."
---

# Round 2 fixer-report

Commits, each green under `lym gate test --upto 1` for the packages it touches:

- `bb29278e2` fabricengine: fix G1, G4
- `a1d8b876c` fabricengine: fix G3, O3
- `3a5e28ea6` fabricengine: fix G2
- `bacdd3236` fabricengine: fix O1, O2
- `1207f0b5e` tools/sandbox: fix G1, G2, G3, G4 (the fabric suite's F23 scenario)
- `3366b8b51` fabricengine: fix gate findings (comment lines)

G1 and G4 share `archiveTagNamespaceAt`, so they are one commit.
O1 to O3 are the review's observations, fixed on the orch brief (creel #2395); O4 to O6 are left alone, as it asks.

## Gate repair

The round's gate refused two comment lines that `bb29278e2` added, each ending in a comma before a clause on the next line, so no test ran:

- `internal/fabricengine/archive.go:226`
- `internal/fabricengine/doc.go:532`

Each is now two sentences, one per line.
`lym loom lint-comments --base 8a8b8debb`, which lints every comment line this round added, answers `ok: true`.
`lym gate test --upto 1 ./internal/fabricengine` passes at `3366b8b51`.
No code or test logic changed.

## Changes

- **G4** (fixed differently, test placement only): `RecordsNamespaceCodeTags` (`code-tags/`) is declared beside `RecordsNamespaceTags` in `recordslayout.go`.
  `archiveTagNamespaceAt` (`archive.go`) returns, for a code store of a one-repository hub, a namespace whose tags prefix is `refs/lym/code-tags/`;
  a records store keeps its own namespace.
  `pushArchiveTag`, `remoteTagsUnder`, `readSideOwnerArchives` and the hand push in the failed-step notice follow it with no further change.
  No fetch refspec of either store maps the new prefix, so the records store no longer fetches code-side tags or commits.
  New test `TestReadOwnerArchives_OneRepositoryHubKeepsEachStoresTagsApart`, over `hubforge.NewOneRepoHub`:
  after an owner archive of both sides, `ReadOwnerArchives` lists each tag once under its own side,
  each store holds its own owner tag alone, and neither holds the other's commit after the read and a `git fetch origin` in the records store.
  The package doc's one-repository section and `docs/getting-started.md` name the prefix.
- **G1** (fixed differently, see frontmatter): the comparison of the two stores' `remote.origin.url` strings is gone from `archiveTagNamespaceAt`.
  `TestDiscardTaskBranch_OneRepositoryHubArchivesUnderTheCodeStoresPrefix` is now a table of two rows:
  the stores spelling origin alike, and the code store's URL respelled as `file://<bare>`.
  Each asserts the tag under `refs/lym/code-tags/archive/<slug>/<sha12>` and no ref under `refs/tags/` or `refs/lym/tags/`.
- **G3** (fixed differently, see frontmatter): `runRemoteTeardown` first calls the new `pushOwedArchiveTags` (`remoteteardown.go`).
  It lists the local `archive/<slug>/*` tags in the records store for a removal or cleanup, and in the code store for a discard and for a removal that deletes the task branch on origin.
  `pushLocalArchiveTags` (`archive.go`) reads origin's with `remoteTagsUnder` and pushes each tag origin lacks with `pushArchiveTag`.
  A failure wraps `ErrArchiveFailed` and stops the job before any delete on origin.
  A store with no origin, or with no such local tag, costs no read of origin.
  `PendingTeardown` now carries `worktree`, `weft_tip` and `task_tip`, and `pendingTeardownWayForward` names the tag push first.
  The wording is shared with `failedRemoteSteps` through `withArchiveTagPush`, `discardWayForward` and `recordsArchiveWayForward`.
  New tests in `remoteteardown_rerun_integration_test.go`, over `hubforge.NewOneRepoHub`:
  - `TestRemove_RerunAfterItsTeardownChildEndedArchivesTheLocalRecordsTip`: a removal whose pair holds a records commit origin lacks, its remote job dropped.
    The pending record's way forward equals the tag push and the re-run, and `Remove` again leaves `refs/lym/tags/archive/<slug>/<local sha12>` on origin with the records branch deleted.
  - `TestDiscardTaskBranch_RerunAfterItsTeardownChildEndedArchivesTheTaskTip`: a discard whose task branch holds a commit origin lacks, its remote job dropped.
    The removal and the discard again delete the task branch on origin with `refs/lym/code-tags/archive/<slug>/<local sha12>` there.
  - `TestPendingTeardowns` gains a record that carries a task tip and asserts its way forward.
  Test seam: `Topology.teardownChildEnds`, set by `SetTeardownChildEndsForTest`, makes the in-process hold record the job as pending and drop it unrun.
  That is the state a detached child killed before its first remote step leaves.
  One consequence beyond the killed-child case: any local `archive/<slug>/*` tag origin lacks is pushed at the slug's teardown, the tags `DropLocal` keeps included.
  `PATTERN-fabric-boundary`, the package doc and the `status` and `prune` help text say so.
- **O3**: `RunTeardownChild` removes the pending record only after a job with no failed step.
  With a failed step it calls `keepFailedTeardown` (`pendingteardown.go`), which writes the failed steps' ways forward into the record;
  `PendingTeardowns` prints a stored way forward as it is.
  `TestRunTeardownChild_SettlesThePendingRecordOnceItEnds` (renamed from `…ClearsThePendingRecordOnceItEnds`) gains a row:
  a job whose origin does not answer keeps the record, with the tag push and the discard's re-run as its way forward.
- **G2**: `Bolt.pullFirst` (`bolt.go:305`) reads `originNamespaceAt(b.path)` before it fetches and returns its error.
  New test `TestBolt_PullThenCommitWritten_RefusesARecordsStoreWithNoNamespacedFetchRefspec`, over `hubforge.NewOneRepoHub`:
  with `remote.origin.fetch` unset, the error names `git -C <board> config --replace-all remote.origin.fetch`, holds no `resolve upstream` text, and no write runs.
- **O1**: `refuseOneURLClone` refuses a plain one-URL clone of an origin that holds records branches and no board.
  It names a seed that stopped before its board landed and `lym fabric clone --single-repo --shortname <shortname> <url>`.
  New row `SeedThatStoppedBeforeItsBoard` in `TestCloneHub_OneURLRefusals`.
- **O2**: `CloneHub` refuses a `cwd` that is not an existing directory, by name, before any git runs.
  New test `TestCloneHub_RefusesAParentDirectoryThatIsNotThere`.
- **Sandbox suite**: `tools/sandbox/SANDBOX-FABRIC-SUITE.md` scenario F23 now drives the code store's prefix with a respelled URL, reconcile with no fetch refspec,
  the re-run after a killed child that held an unpushed records commit, the record a failed step keeps, and the two clone refusals.
  `lym gate test --upto 1 ./tools/sandbox` passes.

## Verification

- Gate: `lym gate test --upto 1 --guards ./internal/fabricengine ./internal/fabriccli` passes at `bacdd3236`, guards included.
  `lym gate test --upto 1` over the branch's diff scope passes at `1207f0b5e`.
  Neither call runs the comment lint, which is why the two lines under Gate repair got through; the lint now runs clean over the round.
- Sabotage proofs: each fix was broken, its test failed at the intended assertion, and the fix was restored.
  - G4: the code store given the records namespace again. The owner-archive test listed each tag twice and found the other store's commit; the discard test found the tag under `refs/lym/tags/`.
  - G1: the URL comparison put back. Only the `file://` row failed, with the tag under `refs/tags/`.
  - G3: the new step skipped. Both re-run tests found no archive tag of the local tip on origin, the discard's with its branch deleted there.
    The tips dropped from the record: the three way-forward assertions failed.
  - O3: the record always removed. The new row found no pending teardown.
  - G2: the layout read's error ignored. The test got `gitrepo: resolve upstream refs/remotes/origin/lym/heads/main: reference not found`, the review's text.
  - O1, O2: each check disabled. The tests got `no recorded warp binding` and `fork/exec /usr/bin/git`, the review's texts.
- Determinism: every new or changed test passed 20 times in parallel (`-count=20`), with no failure.
- Live drive, in a one-repository hub built by `lym harden sandbox new --single-repo` from `1207f0b5e`:
  - G1, G4 (P2): after `git remote set-url origin file://<bare>` in the code store, `lym batten cancel t1 --remove` left
    `refs/lym/code-tags/archive/t1/9997eaaef41a` and `refs/lym/tags/archive/t1/1c49827a851c` on origin and nothing under `refs/tags/`.
  - G2 (P4): with `remote.origin.fetch` unset, `lym fabric reconcile` answered the records-store message with both restore commands; with them typed, it answered its normal envelope.
  - G3 (P6, `k1`): a pair with one unpushed records commit, its `remote-teardown` child killed with `kill -9` while a `pre-receive` hook held the records branch push.
    Origin kept both branches at their old tips, and the commit sat under the local tag `archive/k1/a5ec17eda294` alone.
    `status` and `prune` named the push of that tag to `refs/lym/tags/archive/k1/a5ec17eda294` and then `lym fabric remove k1 --remote`.
    That re-run alone, with no hand push, left the tag on origin beside the tag of origin's old tip, deleted both branches and emptied `pending_teardowns`.
  - O3 (`k2`): with the hook declining `refs/lym/tags/archive/k2/*`, `status` kept `k2` listed with the tag push in its way forward; the re-run with the hook lifted emptied the list.
  - O1, O2: the plain clone of a bare holding `refs/lym/heads/main-weft` and no board named the seed, and `--into <missing directory>` named the directory.
  - `lym harden sandbox clear` ended the drive; `lym harden sandbox list` shows no hub, no sandbox process is left, and `git status` is clean.
- Deviation, as in round 1: the new regression tests require tier 1, not tier 2.
  They run over real git hubs built by hubforge, with the killed child stood in by the Topology seam.
  The `kill -9` scenario itself was driven live as above and is in the suite's F23, not in a tier-2 test.
- Not driven live: G4's read through `lym adopt --archives`, which needs a worktree another machine owns; the hubforge test covers it through `ArchiveOwnerTips` and `ReadOwnerArchives`.
  The killed discard (`k7`) was not driven live either; its hubforge test covers the removal and the discard again.

## For the reviewer

- No migration is written for archive tags an earlier binary pushed elsewhere.
  `main` holds the one-repository layout and pushes a code store's archive tags to plain `refs/tags/`; round 1 moved them to `refs/lym/tags/` on this branch only.
  A code store now reads `refs/lym/code-tags/` alone, so it does not list such older tags; they stay on origin where they are.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
