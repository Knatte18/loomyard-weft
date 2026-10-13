---
format: 5
approved: false
language: go
---

# Plan: reed window order — overview first, agents in start order, shell last

A reed session's visible windows follow one fixed order: the run overview first, one window per agent in start order, then the shell (Selvage) last, with hidden windows after it.
Attach and every automatic switch land on the most relevant running agent, falling back to the overview and never landing on the shell.
The plan builds the shared self-re-exec helper, the new checklist frame and the plain overview panes first, then the watchdog's zoom fix, then reed's window order, the `overview` anchor, the focus rules and the window navigation, and last moves the loom and orch strands onto the new placements.

## Card Index

1 — selfexec-helper — add `internal/selfexec`, the leaf that re-execs a long-lived process on a changed binary
2 — checklist-frame — redraw the loom checklist as detail blocks, row list, state line and a full wait line
3 — plain-overview-panes — drop the overview panes' content-size report and truncation, and re-exec them on a changed binary
4 — reapply-no-zoom-flicker — skip the watchdog re-apply, and its zoom bracket, for the last applied or last failed box
5 — window-order-and-shell — place own and hidden windows around a `shell` window and fix sessions from an older build
6 — overview-anchor — add the one-per-session `overview` anchor that always opens the first window
7 — focus-rules — select the relevant window on attach, on a new agent window and when a viewed window ends
8 — window-navigation-bar — step Alt+Up and Alt+Down across windows and replace VIEW with the `shell` button
9 — strand-placements — move the status and runs strands to `overview` and the driver and orchestrator to their own windows

## Shared Decisions

### Decision: own-window covers the overview

`isOwnWindow` means "a strand with a window of its own" and covers both `own-window` and `overview` from card 6 on, so every liveness, reconcile and resolver path treats an overview pane like an own-window pane; only the placement, the name, the unseen mark, the one-per-session refusal and the focus rules ask `isOverview`.
An agent window, which the focus rules count, is a window holding an `own-window` strand's pane, never the overview's.

### Decision: window positions come from one listing

Every placement and focus decision reads `listSessionWindows` (index, hidden flag, name) beside the existing session pane listing, so window index order is the one order the bar, the Alt keys and the focus rules share; no code sorts or renumbers windows.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
