---
format: 5
approved: false
language: go
---

# Plan: runs-overview

Give the operator one place in the prime's reed session that shows every batten run in flight and its loom stage: a `runs` strand above the orch, sized to its content.
The same task builds reed's strand sizing (a top anchor, a content kind and a sharing kind beside the positional rule), sharing seat strands, the pair's gate checklist strand, a hub notice log, Selvage in the Alt-key cycle, and a run window that closes when its run ends.
The cards run in dependency order: render's vocabulary, then reed's use of it and its content-rows operation, then the shared pane package, then each consumer.

## Card Index

1 — render-size-vocabulary — the top anchor, content and sharing size kinds, ValidateDisplay and their heights in render's policy layer
2 — reed-content-strands — reed persists content rows, plans with the title row, validates displays and splits top strands at the window top
3 — reed-content-rows-op — Engine.SetContentRows, the bounded report that resizes one content strand under the zoom bracket
4 — reed-selvage-cycle — a display-only Selvage marker, Selvage in the Alt+Up/Down cycle and Alt+z on Selvage, in the strands' window only
5 — statuspane-pane — the shared pane loop, frame fitting, content-rows report, two-strike status reader and minute-resolution time format
6 — statuspane-build-identity — move loom's strand build identity and sidecar into statuspane for both pane strands
7 — loom-checklist — the hidden loom checklist verb run in the loom-status strand on both drivers
8 — batten-notice-log — every composed batten notice appended once per process to the hub notice log
9 — batten-overview — the hidden batten overview verb listing the runs in flight
10 — orch-runs-strand — orch start ensures the runs strand above the orch, kept or replaced by build
11 — seat-sharing — seat-table strands declare the sharing kind
12 — run-window-close — the run terminal window closes on a 0 attach or a removed pair

## Shared Decisions

### Decision: physical-top-placement

`render.Rules` resequences cells by the window's physical pane order, so a top-anchored strand is placed at the top by its split, never by the layout string: the first top strand of a window takes a full-width split above every pane, and a later one splits below the lowest top strand (card 2).

### Decision: title-row-once

Each apply and each attach reads the window's title-row answer once and plans both the layout and its pins from it, so a content strand at window row 0 shows its reported line count (cards 1 and 2).

### Decision: pane-reports-its-own-name

A pane process reports its line count with the strand name reed exported to it (`agentname.StrandNameEnv`) and its own `TMUX_PANE`, so reed's name and pane checks bound the report to the strand that runs it (cards 3 and 5).

### Decision: minute-resolution-times

The overview and the checklist render times in whole minutes, so a pane's text changes at most once a minute and its poll backs off between changes (cards 5, 7 and 9).

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
