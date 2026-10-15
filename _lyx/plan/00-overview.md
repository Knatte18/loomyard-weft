---
format: 5
approved: false
language: go
---

# Plan: run-pane-timing

Make the per-run checklist pane tell the operator where a run's time went: closed times to the second, a total per review round, a line per gate entry with its own time and the rework its refusals caused, rework after a gate row's refusal, and a number per entry fixed by the recipe, with the same numbers in batten's runs overview.
The gate times come from spans shuttle records per gate entry, carried by every gated row into its shed history entry.
Separately, every review segment's Bouncer and Burler sessions carry a segment-prefixed role, so reed's strand, window and bar names tell the segments apart.

## Card Index

1 — statuspane-format-seconds — `FormatSeconds` beside `FormatMinutes` for closed spans
2 — shuttle-gate-spans — shuttle's gate outcome reports each entry's gate, held and rework spans
3 — history-gate-spans — shed history entries store gate spans beside `gate_attempts`, folds append them
4 — surface-gate-outcomes — burler's review result and webster's run result carry their gate outcome
5 — carry-gate-spans — every gated row copies its gate count and spans into its history entry
6 — prrework-carry-gate — PR-Rework's Done and Stuck exits keep its session's gate count and spans
7 — recipe-entry-numbers — one rows lookup over every recipe, batten's included, and `EntryNumbers`
8 — checklist-seconds-round-totals — closed times to the second, open ones in minutes, round totals
9 — checklist-entry-numbers — checklist entries numbered in recipe order, a review segment taking one
10 — checklist-gate-lines — a gate line per gate entry under its visit
11 — checklist-gate-rework — a gate row's refusal marks its rework visit and sums its rework
12 — overview-entry-numbers — batten's overview stage cell carries the entry number
13 — role-prefix-config — optional `role_prefix` on the Bouncer, BurlerReview and BurlerFix rows
14 — recipe-role-prefixes — the shipped recipes set `role_prefix` per review segment, pinned by a test

## Shared Decisions

- Every closed span is drawn with `statuspane.FormatSeconds`; every time that includes the open visit with `statuspane.FormatMinutes`, so a pane redraws at most once a minute (D1).
- Gate spans are measured only in shuttle's wait loop, on the run clock, and stored by `shedengine` as a plain stdlib struct of RFC 3339 strings; no two spans of one entry overlap.
- Entry numbers come from `loomrecipe.EntryNumbers` alone, so the checklist and the overview never disagree.
- Roles are passed as role strings; only `agentname`, through reed, forms a session name.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
