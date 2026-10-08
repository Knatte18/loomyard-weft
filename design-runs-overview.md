# Runs overview pane under the orch; batten windows hidden

`runs-overview` · Note · feature, reed, orch, batten

One place for the operator to see every run in the hub and the state it is in.
`lyx shed status [run-id]` is per run and `lyx orch status` covers only the orchestrator, so today the hub polls each run separately and the operator sees run events only through the orch's chat.

## Layout of the prime's reed session (operator, 2026-10-08)

This replaces the 2026-10-05 choice of an own window for the overview.

- The orch runs as today, one strand pane in the prime's strand window.
- The runs overview is its own pane directly under the orch, in the same window.
  It is as small as possible while still showing every run in flight and its shed state.
- VIEW shows both panes; the `orch` strand button zooms the orch alone, so switching to an orch-only view is one key or click.

## Batten windows stay, hidden (operator, 2026-10-08)

`lyx batten run --window` gives each batten run its own window in the prime session, which is where its process lives.
That stays, but the windows are not visible: they carry a marker the status bar skips, so status line 1 shows no `batten:<slug>` buttons.
Their output says nothing useful to the operator; the overview pane is where runs are watched.

## What the overview shows

- One line per run with state, current producer and age.
- Open: whether the running notice log (one line per batten notice, each with the run's attach command) shows in the same pane, or the pane holds only the run lines to stay small.
  Batten appends every notice to a hub log file under the prime's `.lyx` either way.

## Open: the pane's height

The overview's height follows the number of runs, which is what led to the own-window choice on 2026-10-02 and 2026-10-05.
The layout has to size the pane to its lines (reed's told geometry), or fix it at a small height with a summary line when the runs do not fit.
reed's one layout rule today would collapse the orch above and grow the overview, the opposite of what is wanted; note `reed-layout-shares` covers the sizing this pane needs.

## Notices to the orch get shorter

The `[batten notice]` line typed into the orch drops the trailing `cd <pair> && lyx reed attach`: the orch knows where a run's pair is, and the attach command is for the operator, who gets it in the overview.
`renderNotice` in `internal/battenshed/notice.go` builds that line.

## Notice wording: a wait is not an error (operator, 2026-10-05)

Shed keeps the reason for every halt in `Status.Error`, including an awaiting PR-Gate, and `renderNotice` labels it `error:` whatever the state.
Seen: `child left running; child state awaiting, error: the pull request awaits review; …`.
Label the text by state: `error:` only for failed and blocked, and `waiting:` (or the bare reason) for awaiting and paused.
Phrase the condition by what happened, e.g. `board-config-per-worktree: awaiting at PR-Gate: the pull request awaits review`, not `child left running; child state awaiting`.
