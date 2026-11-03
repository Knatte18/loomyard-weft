```yaml
lens: forced-edits
outcome: reported
```

Findings for the forced-edits lens:

1. **BLOCKING** — Card 6, re-sign of `internal/fabricengine#archiveWeftTip`. `ArchiveWeftTipForTest` (`internal/fabricengine/export_test.go:650-651`, `var ArchiveWeftTipForTest = archiveWeftTip`) and its callers still use the old call shape. `TestArchiveWeftTip`, `TestArchiveWeftTip_NoOriginSkips` and `TestArchiveWeftTip_UnreachableOriginErrors` call it with 4 arguments and unpack `(tag, reason, err)`, in `internal/fabricengine/archive_integration_test.go:65,83,101,116,164,197`. The new signature takes 5 arguments and returns `(weftArchive, string, error)`, so these break at compile time, and card 6 lists none of them. The wrapper's comment, "has no verb caller yet", is also stale.

2. **BLOCKING** — Card 10, `internal/battencli#battenCLI.refuseContinuePass` and `internal/battencli#TestBattenPreRun_ContinuePass`. Card 10 calls `taskWorktreeGone` directly inside `refuseContinuePass`, not through the nil-safe `readopt` seam. The row "a running Run-Shed run the state names passes" (`internal/battencli/continue_pass_test.go:48`, setup at :62) only creates `.git` under the prime and no task worktree at `WorktreePath(prime, slug)`. That row therefore reads gone and is now refused, and every row that expects a continuable pass has to create a resolving task worktree. The card only says the test "gains a row" and never mentions changing the existing passing row, so the card gate fails as written.

3. **MEDIUM** — Card 2, `plan:internal/gitrepo#Repo.PushBranch`. The card says each attempt goes through `pushPlainWithRetry`. But `func (r *Repo) pushPlainWithRetry() error` (`internal/gitrepo/push.go:134`) takes no branch, because it pushes the checked-out branch. To push a named branch that is not checked out, it must be re-signed or given a sibling, and no card lists `internal/gitrepo#Repo.pushPlainWithRetry` as an Edit or a re-sign.

4. **MEDIUM** — Card 13, the D3 wiring. The card puts the new seams in `pairteardown`'s `New` and fills the ender's seams "where the ender is built". Neither constructor is listed:
   - `internal/pairteardown#New` is not on the card's Edit list.
   - `EndOwnedElsewhere` builds `ownedElsewhereEnder{…}` (`internal/tendercli/deps.go:269-280`) and is not listed either.

   Without these, production never sets the fields and the warning is never produced.

5. **LOW** — Card 1. The `Repo.PushRebaseFree` doc comment says "A remote "(failed)" or "cannot lock ref" refusal is retried" (`internal/gitrepo/push.go:~151`, read through `lym quarry resolve internal/gitrepo#Repo.PushRebaseFree`). With the third retryable reason it is incomplete, and `internal/gitrepo#Repo.PushRebaseFree` is not on card 1.

6. **LOW** — Cards 3, 4 and 5: `Add`'s contract doc. `Topology.Add`'s doc comment (`internal/fabricengine/add.go:171-202`) describes pre-flight, `AdoptLocal` and the refusals. It is not updated for the tracking-ref drop (card 3), `AdoptOnly` with its typed refusals and the reused records worktree (card 4), or the teardown-lock hold and its wait-out refusal (card 5). Those cards list `Topology.add` but not `internal/fabricengine#Topology.Add`.

7. **LOW** — Card 5, the teardown lock now has more holders. Three comments still describe a lock held only by `Remove` and the detached pushes, and card 5 lists only `doc.go`:
   - the file header comment of `internal/fabricengine/teardownlock.go:1-4`;
   - `TeardownLockPath`'s doc, "excludes a removal … against the pair's detached pushes" (`teardownlock.go:40`);
   - `ErrPairPushRunning`'s doc, "the sentinel Remove's refusal wraps" (`teardownlock.go:23`).

   After card 5, `Add`, the discard and the synchronous pushes also hold or wrap it.

8. **LOW** — Card 1, the text of a lost delete lease. `DeleteRemoteBranchLeased` will return a bare `ErrLeaseRejected`, which drops today's `leased at <sha>` detail. That changes the text of `DiscardTaskBranch`'s error (wrapped at `internal/fabricengine/taskbranchdiscard.go:124`) and of `Add`'s step-11 replace error (`internal/fabricengine/add.go:646`). No test asserts the old text (grep of `_test.go` for "leased at" or "stale info" found nothing), but neither member's message change is stated on card 1.

Tool calls beyond my context:
- Grep for callers of the re-signed helpers and for lease-text assertions in tests: the pre-read covered only production call sites, not test-side drivers.
- `lym quarry resolve` of every existing glyph the cards list: to confirm the spellings. All resolved; one result also gave `PushRebaseFree`'s doc for finding 5.
- Grep of the export wrappers and `PushPairAnchored` test users: my context did not have `ArchiveWeftTipForTest`.
- Read of `destroy.go` around `updateRemoteBranch`, the push-both-sides enforcing test, the ender constructor and `battenrecipe` imports: to check how the lease is handled and how card 13 is wired.
- Read of the `continue_pass_test.go` rows: my context had only the test's name, not whether its passing row builds a task worktree.
- Grep of gitrepo's push methods and the push test fixtures: to check whether `pushPlainWithRetry` can take a branch.
