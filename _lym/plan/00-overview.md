---
format: 5
approved: false
language: go
---

# Plan: lym adopt runs in the foreground over pushed state

`lym adopt <subject>...` stops spawning a detached worker that writes an adopt record.
It runs every step in the caller's process over tracked state: one Bolt commit writes the hold, the owners and this machine's registry entry;
each moved run is taken over from origin with a pushed takeover entry; the catch-up, deploy and workspace follow; the adopt gate runs in the foreground;
a green check clears the hold, and a failing one files a fix task under an `adopt-fix` hold that keeps the try counts.
`--take-hold` moves another machine's hold of the same subjects here and is denied to agents.
The adopt record, the adopt pause, `--keep-local`, `--drop-local` and `--discard` retire, and every reader moves to the Bolt hold.

## Card Index

1 — board-replay-holds — Bolt replay carries a commit's holds and registry, refusing an adopt commit over another machine's new hold
2 — board-adopt-write — the one-commit adopt write with `--take-hold`, and the adopt-fix and clear setters
3 — fabric-takeover-writer — the takeover entry's production writer with its rejected-push retry, the origin run read and the hand-over mark's drop
4 — shellguard-take-hold — a shell-guard row denies `lym adopt --take-hold` to agents
5 — own-hold-reader — the orch watcher and the dobby read this machine's Bolt hold
6 — beam-status-hold — `lym beam status` shows the Bolt hold
7 — batten-hub-run-slugs — the hub-wide run list for the catch-up
8 — adopt-check-ends-on-hold — the check's ends keep tries in the hold and clear it on green
9 — adopt-foreground — the foreground runner, retiring the worker, the record use and the adopt pause
10 — retire-adopt-record — remove `internal/adoptrecord`
11 — retired-adopt-wording — ways forward and docs stop naming the retired flags and pause

## Shared Decisions

- The setters land in boardengine (card 2) and the takeover writer in fabric (card 3) before their production caller in adopt (card 9); the test-only-code guard runs at the plan-level gate, after card 9.
- One fix task stands at a time: the hold's `FixTask` names the first failing subject's task, and `lym adopt check` checks every subject of this machine's hold, so the task's landing re-checks them all.
- A plain adopt's refusal over another machine's hold names waiting or asking the operator, never the take-hold flag, since an agent may read it.

## PATTERN changes

- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until an adopt's check is green.
