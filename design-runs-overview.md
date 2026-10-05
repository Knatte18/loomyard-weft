# Runs overview and event log in their own window in the prime session

`runs-overview` · Note · feature, reed, orch, batten

One screen for the operator: every run in the hub, and a running log of what happened to them.
`lyx shed status [run-id]` is per run and `lyx orch status` covers only the orchestrator, so today the hub polls each run separately and the operator sees run events only through the orch's chat.

## Where it shows

Operator 2026-10-05: its own tmux window in the prime's reed session, beside the orch's window, so the orch pane keeps its full height.
An own window also removes the 2026-10-02 problem of a status strand under the orch whose height would have to follow the number of runs.
With `reed-status-tabs`, a session with more than one window shows window tabs, and Alt+Left/Right switches between the orch and the overview.

## What it shows

- Top: one line per run with state, current producer and age.
- Below: a running event log, one line per batten notice, each with the run's attach command (`cd <pair> && lyx reed attach`), so the operator can jump into a run from it.
  Batten appends every notice to a hub log file under the prime's `.lyx`, and the window tails it.

## Notices to the orch get shorter

The `[batten notice]` line typed into the orch drops the trailing `cd <pair> && lyx reed attach`: the orch knows where a run's pair is, and the attach command is for the operator, who now gets it in the log.
`renderNotice` in `internal/battenshed/notice.go` builds that line; the orch's line and the log's line come from the same notice.
