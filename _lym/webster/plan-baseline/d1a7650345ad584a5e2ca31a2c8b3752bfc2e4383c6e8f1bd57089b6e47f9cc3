# Card 6 — remove-archive-order

**Create:**
- `plan:internal/fabricengine#weftArchive` -> `type weftArchive struct`
- `plan:internal/fabricengine#archiveTagPushHook` -> `var archiveTagPushHook func(tagRef string) error`
- `plan:internal/fabricengine#SetArchiveTagPushHookForTest` -> `func SetArchiveTagPushHookForTest(t testing.TB, hook func(tagRef string) error)`
- `plan:internal/fabricengine#SetArchiveSleepForTest` -> `func SetArchiveSleepForTest(t *Topology, sleep func(time.Duration))`
- `plan:internal/fabricengine_test#TestRemove_ArchiveOrder` -> `func TestRemove_ArchiveOrder(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#archiveTagPushDelays`
- `internal/fabricengine#archiveWeftTip` -> `func archiveWeftTip(rec *Mutations, l *lymcwd.Location, slug, weftBranch string, pushBranch bool, sleep func(time.Duration)) (archive weftArchive, skippedReason string, err error)`
- `internal/fabricengine#tagAndPushArchiveTip` -> `func tagAndPushArchiveTip(rec *Mutations, repoDir, slug, tip, side string, sleep func(time.Duration)) (string, error)`
- `internal/fabricengine#archiveTaskTip` -> `func archiveTaskTip(rec *Mutations, repoDir, slug, tip string, sleep func(time.Duration)) (string, error)`
- `internal/fabricengine#Topology.archiveDiscardedTips`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#Topology.Cleanup`
- `internal/fabricengine#RemoveResult`
- `internal/fabricengine#CleanupBranchEntry`
- `internal/fabricengine#ArchiveWeftTipForTest` -> `func ArchiveWeftTipForTest(rec *Mutations, l *lymcwd.Location, slug, weftBranch string) (tag, skippedReason string, err error)`
- `internal/fabricengine_test#TestArchiveWeftTip`
- `internal/fabricengine_test#TestArchiveWeftTip_NoOriginSkips`
- `internal/fabricengine_test#TestArchiveWeftTip_UnreachableOriginErrors`
- `internal/fabricengine_test#TestOriginWrites_LocalMarkedBranch`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#TestRemoveFields`
- `internal/fabricengine_test#TestCleanup_ApplyArchivesEachOrphanBeforeDeleting`
- `PATTERN.md`
- `pattern/PATTERN-push-both-sides.md`
- `pattern/PATTERN-fabric-git.md`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`

**Uses:**
- `plan:internal/gitrepo#Repo.PushBranch`
- `plan:internal/gitrepo#PushWithRetryReasons`
- `internal/fabricengine#resolveArchiveTip`

**Intent:** Decision record D4, and the archive half of D6's cleanup-sweep rule: a teardown pushes the weft tip before the archive tag that names it, and archives a diverged origin tip too.

`archiveWeftTip` returns a `weftArchive` with `localTag`, `originTag` (empty unless a second tag was pushed), `localTip`, `originTip` (origin's weft tip after any branch push, empty when origin has no weft branch) and `leaseTip`, the origin tip a later delete is leased on (card 7).
Its local-only and no-origin skips are unchanged. After resolving the local tip it:
1. Reads the weft branch's origin tip with `remoteHeadTip` (a branch present only on origin is resolved and fetched by SHA as `resolveArchiveTip` does today, and is its own tip).
   When origin's tip differs from the local tip, it fetches that tip by SHA (`FetchWithArgs(origin, tip)`, as `resolveArchiveTip` fetches an origin-only tip) right after reading it, before any comparison, so steps 2 and 3 compare commits both present in the records repository.
2. With `pushBranch` set and origin lacking the local tip (origin tip empty, or not equal to the local tip and not a descendant of it), pushes the weft branch alone through `gitrepo.Repo.PushBranch` in the records repository: rebase-free, the code branch not pushed, under the remote-refs lock, through the same pause gate the tag push passes.
   A failed branch push is logged at Warn and never fails the archive or the removal; it writes no failed-push record and posts no notice, under the `PATTERN-push-both-sides` change below.
   A landed push makes the local tip origin's tip.
3. Tags and pushes the local tip (`localTag`), then, when origin's tip after step 2 is neither the local tip nor an ancestor of it (diverged, or ahead of the local tip), tags and pushes that tip too (`originTag`); step 1 already made it local, so this step fetches nothing.
4. `tagAndPushArchiveTip`'s push goes through card 1's `PushWithRetryReasons` with the extra reason `missing necessary objects`, so it retries that line within `archiveTagPushDelays`, sleeping through its new `sleep` argument (nil sleeps for real, as `PushWithRetry` does today).
   `archiveTagPushHook`, nil in production, stands in for one push attempt when set, so a test can refuse an attempt with that line.
`leaseTip` is the local tip when step 2's push landed, else the origin tip step 1 read, so an archive tag always covers it.

The sleep is a struct field, never a package variable (`PATTERN-test-speed`): `Topology` gains `archiveSleep func(time.Duration)`, nil in production; `Topology.removePair` and `Topology.Cleanup` pass it to `archiveWeftTip`, `Topology.archiveDiscardedTips` passes it to `archiveTaskTip`, which passes it to `tagAndPushArchiveTip`. `SetArchiveSleepForTest` sets it, beside `SetTeardownWaitForTest` in `export_test.go`.

Callers: `Topology.removePair` passes `pushBranch` true; `Topology.Cleanup` passes false, so the sweep archives both tips of a diverged orphan and pushes no branch.
`ArchiveWeftTipForTest` becomes a wrapper function in `export_test.go` in place of today's re-export var, since its callers sit in package `fabricengine_test` and cannot read `weftArchive`'s unexported fields: it calls `archiveWeftTip(rec, l, slug, weftBranch, false, nil)` and returns `weftArchive.localTag`, the skipped reason and the error, and its comment drops "has no verb caller yet".
So `TestArchiveWeftTip`, `TestArchiveWeftTip_NoOriginSkips`, `TestArchiveWeftTip_UnreachableOriginErrors` and `TestOriginWrites_LocalMarkedBranch` keep their call shape and assertions.
`RemoveResult` gains `OriginArchiveTag string` (`origin_archive_tag`, omitempty) and `CleanupBranchEntry` gains `OriginArchiveTag` likewise; `ArchiveTag` stays the local tip's tag.
`removeFields` adds `origin_archive_tag` when set; `TestRemoveFields` gains that row.

Bound: `Remove` waits on one more network push, inside its exclusive teardown hold; only the teardown itself waits on it.
A pair with no weft origin, a local-only pair and a cleanup sweep push no branch.

PATTERN change, prescribed verbatim, replacing the entry's line in `PATTERN.md`:

```markdown
- `PATTERN-push-both-sides` — Pushing run records from Go: code branch goes along; a push refused after retry leaves a failed-push record and creel notice. A teardown's push of a weft branch it archives and deletes goes alone and only logs a refusal. (test) — [background](pattern/PATTERN-push-both-sides.md)
```

Add to `pattern/PATTERN-push-both-sides.md` a bullet: a teardown's archive step pushes the weft branch alone, with no code branch, before the archive tag that names its tip, and a refusal of that push is logged only, since the removal archives and deletes the branch anyway.
In `pattern/PATTERN-fabric-git.md`'s Unwire-and-teardown bullet on the archive tag, add that `Remove` first pushes the weft branch when origin lacks its local tip, then the tag, and that a diverged or ahead origin tip gets a tag of its own.
Extend `internal/fabricengine/doc.go`'s `archiveWeftTip` paragraph with the push order, the second tag and the retry, and `internal/fabriccli/doc.go`'s remove paragraph with `origin_archive_tag`.

Doc comments this card falsifies, rewritten with it:
- `archiveWeftTip`'s whole doc comment and `archive.go`'s file header: the branch push under `pushBranch`, up to two tags, the `weftArchive` it returns, and the `missing necessary objects` retry beside `(failed)` and `cannot lock ref`.
- `Topology.Remove`'s archive sentences (the weft branch push before the tag, the second tag) and its teardown-lock sentence (the hold now also covers that branch push).
- `Topology`'s `teardownWait` field doc: `Add` and the task-branch discard wait on it too (card 5), not `Remove` alone.
- `archiveTagPushDelays`' sentence that only the pair's detached pushes wait on the teardown lock: `Add`, the discard and the synchronous pair push meet it too.

Test `TestRemove_ArchiveOrder`, `tier1`, one hub per row, each topology's sleep shortened through `SetArchiveSleepForTest`:
- `remote` true, a records commit made after the pair's last push, so origin lacks the local tip; an `update` hook on `h.RecordsBare` refuses any `refs/tags/archive/*` update whose target is not reachable from `refs/heads/<weft branch>` on the bare: the removal succeeds, the tag targets the local tip, the hook accepted it (D4 order criterion), and the task branch on `h.CodeBare` is at its pre-removal tip (no code branch push).
- `remote` false, origin's weft tip moved by another clone so it diverges from the local tip: two archive tags, one at each tip, `OriginArchiveTag` set, the diverged branch still on origin, and a later `Add` of the slug reads the pair not live and forks a fresh pair over the archived leftover.
- `remote` false, origin's weft tip ahead of the local tip (another clone pushed on top of it): no branch push, a tag at each tip, `OriginArchiveTag` at origin's tip.
- `SetArchiveTagPushHookForTest` refusing the first attempt with a `! [remote rejected] <tag> -> <tag> (missing necessary objects)` stderr and passing later ones: the removal succeeds and the tag is on origin.
- An `update` hook refusing `refs/heads/<weft branch>`: the removal succeeds and the archive tag is on origin.
`TestCleanup_ApplyArchivesEachOrphanBeforeDeleting` gains a row: an orphan whose origin weft tip diverges from the local one gets an archive tag at each tip, and no weft branch push is made.

**ImpactSummary:** Moves the archive-tag push onto card 1's `PushWithRetryReasons`, re-signs three private archive helpers whose callers (`removePair`, `Cleanup`, `archiveDiscardedTips`, the test wrapper and its four tests) are all on this card, adds one nil-by-default field to `Topology`, one omitempty field to each of two result types and one envelope field in fabriccli; the PATTERN overview grows by one sentence, well under the headroom line.

**Commit:** `6: A teardown pushes the weft tip before its archive tag and archives a diverged origin tip`
