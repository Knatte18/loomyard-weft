# Orch: count the handoff timeout from the orch's read of the note request

Fixes #845.

The orch watcher's handoff-requested phase timed out at the handoff timeout counted from the post of the note request, or from the request time for a requested cycle.
Creel delivers that mail only at the orch's next idle, so a request queued behind a long turn could use up its whole timeout before the orch saw it.
The phase then returned to idle with `handoff timed out`, often just after the orch had written the note, and no cycle ran.

The timeout now counts from the moment the orch read the note request, the creel `read` journal entry of the request's id.

## What changed

- `orchengine.Creel` gains `MailReadAt(id)`, which answers when the receiver holding a mail read it, zero while it is unread.
  `orchcli`'s `creelSeam` implements it by folding the journal entries for the id and taking the first `read` entry.
  A `reminded` entry that lists the id as unread is ignored.
- `tickHandoff` asks for the read on each tick until creel answers one, records it in the new `State.RequestReadAt` (`request_read_at`), and times the phase out once the unpaused time since the read reaches the handoff timeout.
  The field is cleared wherever `State.RequestID` is: at the cycle start, on `enter`, on `toIdle` and in `ResetForFreshLaunch`.
- The old rule, the earlier of the phase entry and `CycleRequestedAt`, is gone, so a requested cycle is timed from the read too.
  `CycleRequestedAt` is still recorded.
  The idle phase's stale-marker removal still ages a pending request from its request time, so a request whose cycle never starts is still abandoned.
- The `orchengine` and `orchcli` package docs, and the `lyx orch refresh` and `lyx orch distill` help text, describe both timeouts.

## Design decisions

- **An unread request never times out.**
  A request stays unread only while the orch never reaches an idle that creel delivers at, which is the same wait the clearing phase already makes for `/clear`.
  While the orch is busy the watcher's creel hold delays nothing, a removed strand ends the watcher through its strand check, and `lyx orch status` shows the phase.
  A backstop counted from the post would bring the bug back for any turn longer than that backstop.
- **The read time is persisted.**
  A restarted watcher keeps counting from the recorded read, and once the read is known the watcher stops querying the journal.
- **A failed read lookup goes through `creelFailed`**, like every other creel call: it is recorded in `State.Stuck` and holds the timeout until creel answers.

## Tests

- `TestWatcher_HandoffTimeoutCountsFromTheRequestRead` covers three cases: a request read at its post, a request read ten minutes after its post, which gets the full timeout from the read, and a request never read, which never times out.
  The test also checks the recorded `RequestReadAt` and its clearing on the return to idle.
- `TestCycleRequest_AbandonedWithinTimeoutOfRequest`'s note case now has the orch end its turn 50s after the request, and checks that the timeout counts from the read rather than from the request.
- `TestCycleRequest_RestartedWatcherMeasuresFromTheRecordedReadTime` empties the fake journal's reads before the restart, so only the persisted read time can time the phase out.
- `TestCreelSeam_FoldsTheJournal` gains rows for `MailReadAt`: an unread mail with a reminder listing it, the first of two reads, and another mail's read.
