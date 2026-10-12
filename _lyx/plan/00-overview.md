---
format: 5
approved: true
language: go
---

# Plan: reed-multi-window — every spawned agent in a full-size tmux window of its own

Every agent a run spawns gets a detached tmux window of its own, or its producer's shared window, stacked like tabs in the run's terminal window, instead of a pane squeezed into window 0.
Window 0 keeps the driver, its status strand, Selvage, the orch and every strand that names its anchor.
Reed gains the `own-window` placement end to end: the session-wide listing that reads own panes, the launch through the one strand chokepoint, groups shared up to `window_share_max` and split by `window_split`, removal that moves a viewer back to window 0, and a status-bar button marked unseen until selected.
Shuttle then makes `own-window` its default, removes dead own-window strands through its stop paths, the seat engine declares one group per table, and `lyx shuttle open` opens a named session with no turn in its own window.

## Card Index

1 — render-own-window-display — render accepts the own-window display with its window group fields
2 — reed-window-config — reed.yaml's window_share_max and window_split keys, validated at up, resume and add
3 — reed-own-window-presence — reed reads own-window panes through a session listing, never as window 0, and ends orphan own windows
4 — reed-own-window-launch — reed opens own-window strands in detached windows of their own through the launch chokepoint
5 — reed-window-groups — reed shares one window per declared group up to window_share_max
6 — reed-own-window-remove — removing an own-window strand moves its viewer to window 0 and relays out its group
7 — reed-own-window-bar — the status bar shows own windows in their segment color, unseen until selected
8 — shuttle-present-strand-removal — shuttle removes dead own-window strands and keeps a died run's last capture
9 — shuttle-own-window-default — shuttle runs open in their own window by default; the orch names window 0
10 — seat-window-group — seat tables declare one window group sized by their seat count
11 — shell-or-else-exit — the pane shell seam gains OrElse and Exit
12 — claude-open-launch — the provider seam opens a named session with no prompt
13 — shuttle-open-verb — lyx shuttle open opens a named session with no turn in its own window

## Shared Decisions

### Decision: own-window-options-read-by-go

`@lyx_own` and `@lyx_group` are window options Go decides on: reconcile ends only a window carrying `@lyx_own` with no bound pane, and a group window is found by `@lyx_group`, never by name.
`@lyx_unseen` and the window's `@strand_color` are display only, like `@lyx_wait` (`PATTERN-wait-mark-display-only`).
Every option is set by window id, never by window name, since two windows may share a name.

### Decision: empty-window-split-is-width

An empty `window_split` reads as `width`, the template default, so a `Config` built in code without the key keeps working; `width` and `height` are the only other accepted values.
A materialized `reed.yaml` always carries the key once `lyx config reconcile` has run, and until then the layered load supplies the template default.

### Decision: window-zero-listing-stays

`listStrandPanes` keeps listing window 0 only, and every layout, Selvage, split and reap decision keeps reading it.
Own-window panes join only the liveness reads, through `presentPanes` over the session-wide `windowPane` listing, so no window 0 behavior changes.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
