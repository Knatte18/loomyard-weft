# Card 8 — activity-notices

**Edit:**
- `internal/battenshed/notice.go`
- `internal/battenshed/deps.go`
- `internal/battenshed/doc.go`
- `internal/battenshed/notice_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`

**Uses:**
- `internal/shuttleengine/activity.go`
- `internal/shuttleengine/claudeengine/activity.go`
- `internal/shuttleengine/waitmark.go`
- `internal/verifytree/marker.go`

**Intent:**
D5, batten side: the quiet notice is judged from agent activity instead of the status file's modification time, and a new `api-error` notice names an API-error stall.

`battenshed` declares its own neutral reading, so it imports nothing new (`PATTERN-told-geometry`): an `AgentActivity` value per live run (producer label, last activity time, API-error flag and text) and an `InnerRunDeps.Activity func(ctx context.Context) (runs []AgentActivity, waitLive bool, err error)` seam, `waitLive` being whether a Go-side wait marker (verify or shuttle) is live.
`battencli/wire.go` wires it for the task worktree: `shuttleengine.ReadAgentActivity` over the task worktree's shuttle config and anchor with a `claudeengine.NewFromConfig` engine, mapped onto the battenshed type, and `waitLive` from `verifytree.ReadMarker` and `shuttleengine.ReadWaitMarker` as `loomcli`'s `reviewWaitingFor` composes them, reading the marker files and never the `@lyx_wait` pane options (`PATTERN-wait-mark-display-only`).
The seam is read for a running child with a live driver strand, at most once per `notice_probe_s`; an error is a `Warn` and sends nothing.

For a running child with a live driver strand:
- `api-error`: the newest turn end of a live run is an API error and that run has not been active for 2 minutes since (a package constant); the notice names the producer and the error text, cut to one line and 200 characters with `oneLine`, and takes precedence over `quiet`;
- `quiet`: no live run has been active for `notice_quiet_min` and no wait marker is live; the notice says how long the agents have been idle;
- with no live run found, quiet falls back to "no agent activity readable" after `notice_quiet_min` since the later of the child's newest history entry time and `since` (card 7).
With several live runs, quiet needs every run idle and api-error needs any one.
A quiet or api-error episode is keyed by the activity, not the status file: its key carries the newest activity time (for the fallback, the later of the newest history entry and `since`), so the episode ends when an agent is active again; `since` rides on the line as content only.
The driver-dead condition stays first, as today.
Update `conditionText`, the condition constants' comments, `InnerRunDeps.NoticeQuiet`'s doc and the notice section of `doc.go`, stating the limits D5 lists (dead pids skipped, a run that keeps writing or hangs in a live wait is never quiet, both notices informational).

Tests (`notice_test.go`, on the injected clock and a fake `Activity`):
- `TestNotice_QuietOnlyForRunningChildWithLiveDriver` pins quiet by the status file's modification time, which this card replaces, so its rows are rewritten to the activity and not paralleled: a running child whose producer keeps reporting new activity gets no quiet notice however long its status file is unchanged; agents idle for `notice_quiet_min` with no live wait marker give one; a live wait marker suppresses it;
- a newest API-error turn end idle for 2 minutes gives one `api-error` notice naming the producer and the one-line error text; activity afterwards ends the episode, and a later stall is a new notice;
- with no live run, the fallback quiet goes after `notice_quiet_min` since the later of the newest history entry and `since`;
- `wire_seams_test.go`: the `Activity` seam reads a fixture run directory with a live pid and a wait marker.

**ImpactSummary:** the running-child notice conditions of the InnerRun producer change; `InnerRunDeps` gains one optional seam wired once in `battencli`; no other caller.
