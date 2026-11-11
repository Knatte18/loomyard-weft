Task: check the fix set for stale twins and cross-references (fix-consistency lens).

**LOW: `internal/shedverbs/doc.go` (loop.detail paragraph near line 244) and the `runUntilStop` doc in `loopwait.go`**
- The fix added a new behaviour: a dead-loop stop that the waiter reached without a spawn channel waits up to `keeperMarkPolls` × `keeperMarkPoll` for the keeper's exit mark.
- Only the `deadLoopStop` and `awaitLoopProcessMark` comments describe it.
- `doc.go` still says only "How the loop ended is read from the loop log's lines after the loop's last starting mark", and the `runUntilStop` doc still says a free lock beside a pid file "reports a dead loop" with no wait.
- Neither contradicts the code, but a reader of the module doc would not learn that a dead-loop report can be delayed.

I checked the other twins and found them consistent:
- `LoopSpec.Spawn`, `LoopSpec.OnStart`, `doc.go:217-228` and the `spawnLoop` doc now agree on keeper versus loop.
- The `OnExit` comment (`loop.go:93`) still holds as written.
- The "status file now reads" clause is now worded the same in the `deadLoopStop` doc, `doc.go:240` and the code condition at `loopwait.go:522`.
- All three `deadLoopStop` callers pass the new argument.
- `loopwait_test.go:531` (the moved-to-B row) still matches the new condition.
- No other file in the repo mentions the old wording.

Tool calls:
1. One `grep` over `*.go` and `*.md` for "now reads", "never runs it" and "deadLoopStop", to find stale twins elsewhere in the repo. My context had only the edited files.
