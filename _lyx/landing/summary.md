# Orch and batten: compact reload, event-driven notices, batten windows

The orch no longer waits out the handoff timeout after a compaction, hears each run stop once and within seconds, and sees one tmux window per batten in flight.

## What changes

- **Compact cycle reload.**
  `orchengine` now decides a compaction landed by asking `CompactedSince` for the boundary, so an assistant turn after the boundary (a message queued during `/compact`, answered at once) no longer hides it and the cycle no longer times out.
  A same-mode cycle request queued while a cycle runs is removed as satisfied when the cycle's action takes effect: at the compaction boundary for compact, when `/clear` is confirmed typed for clear.
  A request of the other mode, or a compact request made after the boundary, stays pending.
- **Run-Shed waits on the child.**
  The `InnerRun` row waits inside its call instead of sleeping one interval and returning `Stuck`.
  It returns on a state change, the arm's own event, a requested pause or a failed stat of the child's status file; every `Stuck` it returns is budget-exempt.
  Batten's history now holds only the child's state changes, a requested pause and a failed status read.
  `poll_interval_s` is 2 in `batten-recipe.yaml`, `notice_probe_s` (30) is new and bounds every read that costs a process or a multiplexer round trip, and the 12-hour running watch window and `max_bounces: 1440` are gone.
- **One notice per stop.**
  Batten sends the state-changed notice from inside the wait, once the driver's stop report exists, the driver ended, or 3 minutes passed.
  A `done` notice goes on the first check that sees it, with no report, because Worktree-Teardown removes the path right after.
  Every notice now carries `since <time>`, `history <n>` and, for a parked stop, `report <path>` or `report none yet`.
  A notice counts as sent only when `QueueNotice` queued it; with no orch strand recorded it is retried per `notice_probe_s`, and a failed queueing is retried at most 3 times.
- **Quiet from agent activity, and an `api-error` notice.**
  `shuttleengine` gains a provider-neutral reading of a worktree's live runs (last activity, and whether the newest turn end is an API error); `claudeengine` supplies the transcript part, recognising Claude's `isApiErrorMessage` marker.
  Quiet (`notice_quiet_min: 45`) is judged from that activity and a live verify or shuttle wait marker suppresses it.
  An API-error turn end followed by 2 minutes of silence sends an `api-error` notice naming the producer and the error text.
- **Driver messages its parent less.**
  The driver stencil renders its notify rule from a `batten-watched` marker (path declared in `shedrun`), which batten writes only while its notices and its driver's parent are the same orch with a recorded strand.
  A watched driver messages the parent only for an `awaiting` stop's `parent_notice`, a question it cannot settle, or a `done` stop whose friction failed.
  An awaiting stop with a `parent_notice` is announced by batten itself only when the driver ended, no report came in 3 minutes, or the child is still undecided 10 minutes after the report.
- **`lyx batten run <slug> --window`.**
  It opens a detached `batten:<slug>` tmux window in the orch's reed session through a new reed `OpenWindow` op, returns at once, and closes the window when batten exits.
  The orch role stencil now starts battens with it instead of `setsid nohup`.
- **Reed addresses its strand window by id.**
  One seam, in `windowtarget.go`, resolves the window of the Selvage pane (or of a present strand pane) and replaces `exactSessionWindowTarget`, so a batten window never gets read as the strands' panes or reaped.
  `new-window` joins the verbs the capability probe requires.

Docs land with the code: `doc.go` of `orchengine`, `battenshed` and `reedengine`, the batten verb text, the recipe header comment, `docs/overview.md`, and `PATTERN-pane-binary-resolution`.

## Design decisions to check

- A 2-second stat poll with a decode only when the modification time changed (and at every `notice_probe_s` regardless) is the event mechanism; there is no file-notification dependency.
- `since` comes from the status file's modification time at first sight of the state, because the child's history has no time for a pause or a producer failure.
- The watched marker, not the seed, carries the driver's rule: the seed is write-once and compared for equality against the child's bootstrap.
- A batten started from a worktree other than the prime writes no marker, so its driver keeps today's rule; a stop may then reach the orch twice, never not at all.
- The batten window sets `remain-on-exit off` in the same invocation that creates it, targeting the window by exact name, and reads the option back by id; a window that does not read `off` is killed and the verb refuses naming `lyx batten status <slug>`.
- The tmux window lives only as long as the reed session, so a tmux restart now kills a batten that the earlier `setsid nohup` start survived.
- D1 and the quiet and API-error notices rely on Claude Code's internal transcript format (`compact_boundary`, `isApiErrorMessage`).

## To check by hand

The plan-level verify was not run in this session.

- Run `go test ./...` and the `tmux`-tier tests.
- Start `lyx batten run <slug> --window` from the prime and confirm the window appears detached, the strands' window keeps `remain-on-exit on`, and the window closes when batten exits.
- Run a compact cycle with a message queued during `/compact` and confirm it reloads without a `compaction timed out` abort.
- Confirm a halted run produces one notice with `since`, `history` and a report path, and that the driver sends no "run stopped" message.
- `cmd/lyx/testdata/test-budget.yaml` changed to fit the new tests, and `internal/testkit/envkit/envkit.go` changed too; check both budgets.
