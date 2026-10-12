# reed: runs overview pane above the orch; Selvage reachable by key

`runs-overview` · Note · feature, reed, orch, batten

One place for the operator to see every run in the hub and the state it is in.
`lyx shed status [run-id]` is per run and `lyx orch status` covers only the orchestrator, so today the hub polls each run separately and the operator sees run events only through the orch's chat.

## Layout of the prime's reed session (operator, 2026-10-08)

This replaces the 2026-10-05 choice of an own window for the overview.

- The orch runs as today, one strand pane in the prime's strand window.
- The runs overview is its own pane directly under the orch, in the same window.
  It is as small as possible while still showing every run in flight and its shed state.
- VIEW shows both panes; the `orch` strand button zooms the orch alone, so switching to an orch-only view is one key or click.

## Layout revised (operator, 2026-10-12)

This replaces the 2026-10-08 placement under the orch; the rest of that section stands.

- The overview pane sits directly above the orch, in the prime's strand window, and VIEW shows both; it is a pane, not a window.
- Its height is dynamic: one line per run in flight, growing and shrinking as runs start and land.
- Each line names the run and the loom stage it is in.
- The batten windows are gone from the prime's reed session, so today nothing shows every run at once.

## What the overview shows

- One line per run with state, current producer and age.
- Open: whether the running notice log (one line per batten notice, each with the run's attach command) shows in the same pane, or the pane holds only the run lines to stay small.
  Batten appends every notice to a hub log file under the prime's `.lyx` either way.

## Open: the pane's height

The overview's height follows the number of runs, which is what led to the own-window choice on 2026-10-02 and 2026-10-05.
The layout has to size the pane to its lines (reed's told geometry), or fix it at a small height with a summary line when the runs do not fit.
reed's one layout rule today would collapse the orch above and grow the overview, the opposite of what is wanted; note `reed-layout-shares` covers the sizing this pane needs.

## Selvage reachable by key (operator, 2026-10-12; merged from `selvage-nav`)

In every reed session, Selvage is reachable like a strand: a free terminal tied to reed is useful, so the operator wants to step to it and zoom it in.

- Today (`internal/reedengine/doc.go`) Alt+Up and Alt+Down step through the strand panes and pass over Selvage, and Alt+z does nothing in Selvage, so Selvage is reached only by mouse and never full screen.
- Alt+Up and Alt+Down include Selvage in their cycle, keeping the zoom, so stepping onto it while zoomed shows it full screen.
- Alt+z zooms and unzooms Selvage like a strand pane.
- Open, for later (operator): whether Alt+Up and Alt+Down should ever step between windows rather than panes.

## Related entries

- Task `batten-notice-trim` drops the attach clause from the notices to the orch; the overview is where the operator gets each run's attach command.
- Task `batten-windows-hidden` hides the batten windows, so the overview pane is where runs are watched.
