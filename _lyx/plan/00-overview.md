---
format: 5
approved: false
language: go
---

# Plan: run-robustness

A run record a newer `lyx` writes must never halt or lose data in a long-lived process on an older binary from this one on: `internal/state` reads every Go-written JSON record with unknown fields accepted and keeps them on every rewrite, the two hand-rolled strict decoders turn tolerant, and a PATTERN entry with a scan pins it.
A verify step that hung is never re-run unfixed by webster's gate, and `lyx gate test` SIGQUITs a Go test binary that keeps a core busy, so its stacks reach the log long before go test's timeout, while every verify gate names that quit and its way forward.
The plan runs against the tree with `board-field-drop` merged, so `board.json`'s own Marshalers are pinned first and stay opaque to the new merge.

## Card Index

1 — board-encoding-pin — pin board.json's Save bytes before state starts keeping keys
2 — state-tolerant-read — ReadJSONStrict accepts unknown fields; tests and comments that relied on the refusal follow
3 — state-keep-unknown — every state write keeps the old file's unknown keys by D2 and D3
4 — state-rename — rename ReadJSONStrict to ReadJSONNoMkdir across the module
5 — shedrun-seed-tolerant — seed.json decodes tolerantly and an agreeing seed keeps its unknown keys
6 — batten-cleanup-tolerant — batten cleanup judges a status with an unknown key
7 — pattern-tolerant-records — PATTERN-tolerant-records, its background and its scan, under the headroom line
8 — gate-busy-quit-key — busy_quit_sec in the hub-wide gate config and Limits
9 — proc-tree-cpu — proc samples a process tree's CPU time and walks ancestry, Linux only
10 — gate-test-watchdog — lyx gate test SIGQUITs a CPU-busy Go test binary
11 — webster-hang-gate — webster's verify gate skips the rerun after a hang and names it
12 — loom-watchdog-halt — loom's verify gate names a watchdog quit and busy_quit_sec
13 — landing-watchdog-halt — the landing verify gate names a watchdog quit and busy_quit_sec

## Shared Decisions

- The watchdog line is rendered and found only through `internal/gateslot`'s `WatchdogLine` and `FindWatchdogQuit`, and the operator's way forward after a quit is its `BusyQuitWayForward`; gatecli prints the line, and webster, loom and landing read it.
- A test that pinned a refusal or a drop D1 to D4 reverse is deleted or flipped in the card that reverses it, never left to the plan-level gate.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
