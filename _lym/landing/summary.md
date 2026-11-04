# Landing notice survives a late request, a down dobby and an absent orch

`landing-watch` left three known limits on the landing notice dobby sends when a task with `notify_on_land` lands.
This change closes all three.

## Behavior

- **A request set after the run landed is answered at once.**
  `lym board upsert` on a done entry stores `notify_on_land` and returns a notice saying the run already landed, and that the parent's dobby sends the line at its next tick.
  The dobby then sends it, since the board write wakes its watch.
  A bare `{"slug":…,"notify_on_land":true}` on a slug the board no longer holds is refused, naming `lym batten status <slug>`, instead of filing a new note.
- **A landing dobby missed is still noticed.**
  Board prune keeps a done entry while it carries `notify_on_land`, so the request outlives Land's prune.
  Each tick, dobby takes up every done entry whose parent is its worktree and that carries the field with no memory of its own.
  It sends `[dobby notice] <slug>: landed as <commit>` when the run is still listed past Land, and `landed, commit unknown` otherwise.
  This covers a landing made while dobby was down, a fresh state record, and a request raised after the last tick that saw the run ahead of Land.
- **A line the orch cannot take waits.**
  Creel now accepts a post to an orch's fixed name of this hub, `<shortname>:orch` or `<shortname>:<slug>:orch`, whether or not a strand of that name is listed.
  The mail stays unread in the pool and is delivered once the orch's strand is back.
  On the dobby side, a decided line is kept in the state record as owed and posted at every tick until a post succeeds.
  The old three-attempt cap is gone.
  An owed line survives the run's listing ending, a restart and a re-exec.
- Once the line goes out, dobby clears the entry's `notify_on_land`.
  When no line is owed, as under `land_notice: off`, dobby clears it at once.
  Either way the next prune can drop the landed entry.

## Design decisions

- **The board holds the request across the landing.**
  The decision record asks for "board done state, not only its own state record".
  Land prunes every done entry, so the request was lost with the entry.
  Keeping a flagged done entry until dobby clears it makes the board the durable record, and dobby needs no history of what it saw.
- **One memory, two sources.**
  `LandNoticeState` gains `Line` (the owed notice) and `ClearOwed`, and loses `Attempts`.
  A run seen ahead of Land is decided at Land as before, counting a request read that tick as well as the remembered one.
  A done entry with no memory is decided by the new board step.
  A slug with a memory is never taken up by the board step, so a run gets one line, not two.
- **`off` now clears the request.**
  Before, `off` left the request on an entry that Land then pruned anyway.
  Now prune keeps flagged done entries, so leaving the field would keep the entry on the board forever.
  The `OffSendsNothingAndClearsTheRequest` row pins the new behavior.
- **Creel's rule is narrow.**
  Only this hub's plain orch names pass when unlisted: an orch thread (`orch-N`), another hub's shortname and any other role are still refused as unknown receivers.
  The orch name comes from `orchroute.StrandOf`, which already answers the fixed name when reed no longer holds the strand.
  A prime orch that was never recorded still posts nothing; dobby keeps that line owed and retries it.
- `battencli.DobbyTasks` now reads the parent worktree of every entry carrying `notify_on_land`, so a done entry reaches only its parent's dobby.
  `boardengine` exports `StatusDone` and `IsDoneStatus` for the dobby and CLI checks.

## Tests

- `internal/dobbyengine/landnotice_test.go`, tier 0: new rows cover done entries with and without a listed run, under `off`, under another worktree, a request raised as the run lands (one line), and owed lines posted until they succeed and after the run leaves the listing.
- `internal/creelengine/verbs_test.go`, tier 0: a post to an unlisted orch of the hub is stored and read by the strand once it is listed; another hub's orch and an orch thread are still refused.
- `internal/boardengine/board_test.go`, tier 0: prune keeps a flagged done entry and drops it once cleared.
- `internal/boardcli` `TestCLI`, tier 1: the upsert notice on a done entry, prune keeping it, and the refusal on a pruned slug.
- `internal/battencli/dobbyreading_test.go`, tier 0: a done entry carrying the field gets its parent read.

## End to end against a live orch

The record asks for the notice to be shown once against a live orch, in a test or by hand.
That has not been done.
The tests above drive dobby over fakes and creel over its fake delivery, not a live orch.
A live run needs this branch's binary deployed, because the creel daemon and dobby both change.

To check by hand after deploy:

1. Stop the prime orch with `lym orch stop`.
2. Set `notify_on_land` on a running task and let it land.
3. Confirm with `lym creel log` that the `[dobby notice] <slug>: landed as …` mail is stored for `<shortname>:orch`.
4. Start the orch and confirm the line arrives and the entry's field clears.
5. Run `lym board prune --apply` and confirm it drops the entry.
