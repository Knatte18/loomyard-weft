# Decision record — orch handoff timeout counts from delivery

## Goal

The orch watcher's handoff-requested phase times out at the handoff timeout counted from when the orch read the note request, so a request queued behind a long turn still gets the full time.

## Scope

- `internal/orchengine`: the `Creel` seam gains a call answering when a posted mail was read;
  `tickHandoff` counts the handoff timeout from that read, recorded in `State`;
  the package doc's handoff-requested description changes to match.
- `internal/orchcli`: `creelSeam` implements the new call by folding creel's journal `read` entries for the mail id;
  the `refresh` and `distill` help text and the package doc describe the two timeouts a request meets.
- Out of scope: the idle phase's stale-marker removal, which still ages a pending request from its request time, and every other phase's timeout.

## Decisions

### D1: The note phase's clock starts at the orch's read of the note request

Decision: `tickHandoff` times the phase out once the unpaused time since the note request's creel `read` event reaches the handoff timeout, for every trigger.
The earlier rule, the earlier of the phase entry and the request time, is dropped.
Rationale: creel delivers the request at the orch's next idle, so the read is the moment the session can start the note; the time it sat queued behind a turn is no time the session had.

### D2: While the note request is unread the phase does not time out

Decision: a handoff-requested phase whose request creel has not journaled as read keeps waiting, as the clearing phase already waits for creel to type `/clear`.
Rationale: a request stays unread only while the orch never reaches an idle creel delivers at.
Bound: this lets the phase, and the watcher's creel hold, last as long as one orch turn or a stall creel reports;
the hold only orders the watcher's deliveries first and delays nothing while the orch is busy, a removed strand ends the watcher through its strand check, and `lyx orch status` shows the phase meanwhile.

### D3: The read time is looked up through the creel seam and persisted in State

Decision: `Creel` gains `MailReadAt(id)`, answering the time of the mail's first `read` journal entry, zero while unread.
The watcher asks it on each handoff tick until it answers a time, then records it in a new `State.RequestReadAt`, cleared wherever `State.RequestID` is.
A failed lookup goes through `creelFailed`, holding the phase, like every other creel call.
Rationale: persisting the read time keeps it across watcher restarts and saves the journal query once it is known.

## Constraints

- `PATTERN-tolerant-records`: the new State field is an added JSON key, read as zero from an older state file.
- `PATTERN-orch-pane-single-writer` and `PATTERN-creel-store`: the watcher reaches creel only through its `Creel` seam; the journal fold lives in `orchcli`.
- `PATTERN-daemon-wakeups` and `PATTERN-log-levels`: the read lookup runs inside the existing tick and logs the found read once, at Info.
- `PATTERN-test-speed`: tests use the injected fake clock and the recording fake creel.

## Auto-mode assumptions

- A darn run holds no interview; every call here is the writer's own.
- The `read` event creel journals for a mail, by a mailbox read or by a typed notice, is the delivery the board entry names.
- A request posted again after a restart has a new id and starts unread.

## Open risks

- An orch stuck in one turn for hours keeps the watcher in handoff-requested for as long, where the old rule returned it to idle after the handoff timeout; the next cycle would have waited on the same turn.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - failed-mailreadat-lookup-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- A note request read long after its post gets the full handoff timeout from the read: it times out only once the unpaused time since the read reaches the timeout.
- A note request never read does not time out.
- A requested cycle's phase no longer times out from the request time; it counts from the read.
- A restarted watcher keeps counting from the recorded read time.
- A hub pause after the read is still discounted.
- `creelSeam.MailReadAt` answers the first `read` entry's time for the id and zero when the journal holds none.
