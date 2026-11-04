```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or none (lens code-tests).

**Behaviour → pinning test**

1. Creel accepts a post to an unlisted orch of this hub (3-segment form), refuses another hub's orch and an orch thread.
   - Pinned by `verbs_test.go` row `Post_ToAnOrchOfTheHubWithNoStrandWaitsForItsStrand`.
   - The 2-segment `<shortname>:orch` form (the prime's) has no row. The row uses only `ly:gone:orch`.
   - `checkReceiverLocked`'s second caller (`verbs.go:840`) has no unlisted-orch row.
2. Mail to an unlisted orch stays unread until its strand is listed, never expired.
   - **None.** The row builds a bare `Store` and lists the strand by hand; no scheduler or sweep runs.
   - `sweep.go:134-149` treats a holder absent from the strand list for two sweeps (30s, then 10s) as gone.
   - `sweep.go:184` includes the unlisted orch as a holder, because `isStrandName` is true for an orch name.
   - `sweep.go:222-224` then journals the unread message undelivered when its sender is a program such as dobby.
   - `TestSweep_RemovedHolders` (`deliver_test.go:1398`) has no unlisted-orch case.
3. Prune keeps a done entry carrying `notify_on_land`, and drops it once cleared.
   - Pinned by `board_test.go` rows "prune keeps a done entry carrying it" and "prune drops the done entry once it is cleared", via the new `GetTask` assertion.
   - Not pinned: a flagged done member of a done project, and a survivor's `depends_on` or `order` naming a kept entry.
4. `lym board upsert` on a done entry prints the "already landed" notice.
   - Pinned by `stepNotifyOnLandOnALandedEntry` (exact notice text).
   - No negative row: an upsert that leaves the flag set on an already flagged done entry, without touching it, also prints the notice.
   - No negative row for a non-done entry.
5. The refusal of a bare `{"slug", "notify_on_land": true}` on an absent slug.
   - Pinned by the same step: error text, and nothing filed.
   - Not pinned: `len(fields) != 2` with an extra key such as `id`, `notify_on_land: false`, and an empty slug.
6. `DobbyTasks` reads the parent of a done entry carrying the field, and not its owner.
   - Pinned by the new `landed` row in `dobbyreading_test.go` (Parent set, `OwnedElsewhere` false).
   - Not pinned: a done entry without the field still gets no parent (only `draft` is asserted).
   - Not pinned: `parents.Of` failing for a done flagged entry, which aborts the whole read at `dobbyreads.go:288-291`.
7. A request set after the run landed, or a landing missed while down, is noticed (`takeDoneEntries`).
   - Pinned by `ADoneEntryWithNoRunSeenIsNoticedAndCleared` and `ADoneEntryWithItsRunFirstSeenPastLandNamesTheCommit`.
   - Not pinned: a listed done run with `Landed` false (`ended without landing` line), and a listed done run that is not `PastLand`.
8. Only this worktree's dobby takes up the entry: pinned by `ADoneEntryOfAnotherWorktreeIsLeftToItsDobby`.
9. One line when the request is raised as the run lands: pinned by `ARequestRaisedAsTheRunLandsIsSentOnce`.
10. `off` sends nothing and clears the request: pinned by `OffSendsNothingAndClearsTheRequest` and `ADoneEntryUnderOffIsClearedWithNoLine`.
11. An owed line is posted every tick with no cap, and outlives the run's listing.
    - Pinned by `AFailedPostIsOwedUntilItGoesOut`, `AnOwedLineOutlivesTheRunsListing` and `AFailedPostOfADoneEntryIsOwedUntilItGoesOut`.
    - A restart or re-exec carrying the owed line, which the summary claims, has no row. No test builds a second daemon over the same state record.
12. A failed `ClearNotifyOnLand` keeps the clear for the next tick and does not repost.
    - **None.** The harness clear always succeeds.
    - The post-then-clear ordering and `memory.Line = ""` at `landnotice.go:169-176` are unpinned.
13. `Deps.ClearNotifyOnLand == nil` drops the memory (`landnotice.go:171`): **none**.
14. A failed `beam.yaml` read (`modeRead` false) decides nothing and takes up no done entry: **none**. Only the board-read failure has a row (`AFailedBoardReadKeepsTheFirstReading`).
15. An unreadable state record in `settleLandNotices` (`landnotice.go:152-155`): none; minor.
16. Memory is empty once settled: pinned by the final assertion in `TestLandNotices`.
17. The notice shown end to end against a live orch, which the Goal demands.
    - **None.** No test drives dobby, creel and an orch strand together.
    - The `Notice` seam in `dobbydeps.go` and creel are never joined in a test.

**Findings**

- **BLOCKING / design** — `creelengine/verbs_test.go` (new row) and `creelengine/sweep.go:134-149,184,222-224`.
  - The "waits for the orch's strand" behaviour has no row over the sweep.
  - By my reading of the code, the sweep returns the unread mail to its program sender, so it is journaled undelivered about 40s after the post.
  - That contradicts Goal item 3 and the summary's "stays unread … delivered once the strand is back".
  - Evidence: `sweep.go:36-48` (loop runs while a strand holds unread mail), `sweep.go:222-224`, `deliver_test.go:1398`.
- **MEDIUM / scope** — no test joins dobby, creel and an orch strand.
  - The Goal's end-to-end demand is unmet by test, and the summary admits it was not done by hand either.
  - Evidence: `landnotice_test.go:70-141` drives fakes only, and `verbs_test.go:202-211` has no dobby.
- **MEDIUM / design** — `landnotice.go:169-177`, `landnotice_test.go:86-90`.
  - No row fails `ClearNotifyOnLand` or sets it nil.
  - The no-repost-after-failed-clear and retry-clear behaviours, which the focus directive names, are unpinned.
- **LOW / design** — `landnotice.go:35-37,100-103`, `landnotice_test.go:74`.
  - No row makes `LoadConfig` fail, so the `modeRead` gating in `rememberRuns` and `takeDoneEntries` is unpinned.
- **LOW / design** — `boardcli/cli.go:1226-1234`, `verbs_test.go:195-218`.
  - No row for an upsert that leaves `notify_on_land` alone on an already flagged done entry. `landedNotice` claims the run "already landed" there.
  - No negative row for a non-done entry.
- **LOW / design** — `boardcli/cli.go:1214-1224`.
  - Refusal triggers `len(fields) != 2`, `notify_on_land: false` and empty slug have no rows. Only the happy refusal is pinned.
- **LOW / design** — `battencli/dobbyreads.go:287-292`, `dobbyreading_test.go`.
  - No row for `parents.Of` failing on a done flagged entry, which aborts every start of the tick.
  - No row for a done unflagged entry still carrying no parent.
- **LOW / design** — `creelengine/verbs_test.go:202`, `pool.go:211-214`.
  - The unlisted `<shortname>:orch` (prime) form has no row, and neither does `verbs.go:840`'s `checkReceiverLocked` caller.
- **LOW / design** — restart and re-exec survival of an owed line, claimed in `summary.md` and `doc.go`, has no test.
- **LOW / consistency (test economy)** — `landnotice_test.go` row `AFailedPostIsRetriedOnTheNextTick` (failPosts 1) is covered by `AFailedPostIsOwedUntilItGoesOut` (failPosts 4, which retries) and `AnOwedLineOutlivesTheRunsListing` (failPosts 1, wantPosts 2). Fold or delete it.

**Tool calls:** none; the inherited context already held everything needed.
