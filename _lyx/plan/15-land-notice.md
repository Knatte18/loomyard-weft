# Card 15 — Every landing sends a landed notice

**Edit:**
- `internal/battenshed#readyMarkerLine`
- `internal/battenshed#rowNotifier.ready`
- `internal/battenshed#landProducer.Call`
- `internal/battenshed#landFixture`
- `internal/battenshed#landFixture.deps`
- `internal/battenshed#TestLand_ReadyNotice`
- `internal/battenshed#TestLand_Dispositions`
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Intent:**
D16: the Land row's last step queues `<prefix> <slug>: landed` after the board prune on every landing, appending `; newly ready: a, b` when the landing made tasks ready, since the alignment agent lands tasks with no orch turn and the notice is the orch's only prompt that a task landed.

`landProducer.Call` calls `p.notifier.ready(ctx, p.slug, ready)` whatever `ready` holds, in place of only when it is non-empty; the step still runs after the prune and the ready read, so a landing that fails before the ready read sends no landed notice and a failed step sends only its failure notice.
`rowNotifier.ready` composes `[batten notice] <slug>: landed` for an empty `slugs` and `[batten notice] <slug>: landed; newly ready: <slugs joined by ", ">` otherwise, keeps its once-per-batten-run guard on the row's marker line, and its doc comment says it is the landing notice.
`readyMarkerLine` keeps its literal `ready`, now meaning "the landing notice was queued", so a Land row resumed across the upgrade after a ready notice was queued sends no second notice; its comment says so.
A notice that cannot be queued, because the orch state is unreadable or no orch strand is recorded, is appended to the hub notice log and then held back by `rowNotifier.send`, which already logs Warn naming the slug and returns unqueued; it is not retried, since the row is Done, and the orch learns of the landing from its next-wake board checks (card 16).
`origin/prod` plays no part here; it is the deploy-recovery baseline only.

`battenshed/doc.go`'s Land paragraph says the row queues one landing notice per batten run, naming the tasks it made ready when there are any, and the rows-after-Run-Shed paragraph no longer implies Land notices only on failure.
`docs/overview.md`'s batten account says `Land` notices the orch of every landing, with the tasks it made ready, and its line that `Land` and `Worktree-Teardown` notify only on a failure or a newly ready task says Land also notifies every landing.

Tests:
`landFixture` gains `orchUnrecorded bool`, which `deps` answers through a `Notices.OrchStrandRecorded` seam.
`TestLand_ReadyNotice`'s "ReadiesNothing" row wants `[batten notice] myslug: landed`, and its re-entry check then pins that a resumed row sends no second one;
a new row "NoOrchStrandWarnsAndQueuesNothing" sets `orchUnrecorded` with a ready task and wants no queued notice and one `level=WARN` `row notice held back; no orch strand is recorded` line naming `myslug`, so the test captures the logger at Warn per row and drops its `t.Parallel` calls.
`TestLand_Dispositions` wants exactly one notice, `[batten notice] myslug: landed`, on every row, in place of none.
`TestLand_FailingStepsAreStuckWithOneNotice` already pins that each failing step queues only its failure notice and that the recovered "ReadyTasksErrorAfterPrune" re-entry sends `landed; newly ready: b`, and needs no change.

**ImpactSummary:** the Land row now queues one notice on every successful landing where it queued one only for newly ready tasks; the line format, marker, retry bounds and failure notices are unchanged, and the orch side lands in the next card.

**Commit:** 15: battenshed: send a landed notice on every landing
