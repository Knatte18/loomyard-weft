---
format: 5
approved: true
language: go
---

# Plan: lifecycle-waits

The orch and batten side of a run's lifecycle stops costing time and sending noise.
An orch compact cycle reloads as soon as its compaction lands, and a same-mode request queued during a cycle is satisfied by it.
Batten's Run-Shed row waits on the child inside its call instead of ticking every 30 s, sends each stop's notice once, timed on the driver's stop report and carrying how old the state is, judges quiet from agent activity and names an API-error stall.
A batten-watched driver messages its parent only with a relay, a question or a failed friction, and the orch starts each batten in its own tmux window of its reed session.
The decision record's D1–D7 map onto the cards below.

## Card Index

1 — orch-compact-boundary — the compacting phase completes on a boundary `CompactedSince` finds (D1)
2 — orch-satisfied-cycle-request — a same-mode request queued during a cycle is removed when the cycle's action takes effect (D2)
3 — driver-notify-rule — the watched-marker path in `shedrun` and the driver stencil's notify rule rendered by it (D6, render)
4 — shuttle-agent-activity — a provider-neutral reading of a worktree's live shuttle runs' activity and API errors (D5, data)
5 — run-shed-waits — the Run-Shed row waits on the child in-call: state change, arm event, pause, stat failure (D3)
6 — batten-watched-marker — batten writes or removes the watched marker for the run (D6, batten)
7 — state-change-notices — state-changed notices timed on the stop report, with `since`, `history` and `report` (D4)
8 — activity-notices — quiet judged from agent activity, and the `api-error` notice (D5, batten)
9 — reed-strand-window — reed addresses its strand window by id through one seam (D7, reed)
10 — reed-window-op — a reed op that opens a detached named window running `lyx` (D7, reed)
11 — batten-run-window — `lyx batten run <slug> --window` and the orch stencil's start step (D7, CLI)

## Shared Decisions

### Decision: battenshed-stays-neutral

`battenshed` imports only `shedengine` and `logger`: every new reading (pause flag, stop report, orch strand, watched marker, agent activity) arrives through an `InnerRunDeps` or `SeedChildDeps` seam wired in `battencli/wire.go`, with `battenshed`'s own neutral types (`PATTERN-told-geometry`).

### Decision: rate-of-reads

Inside the Run-Shed wait, file stats and small file reads run on every `poll_interval_s` check; anything costing a process or a multiplexer round trip (reed status, agent activity, wait markers, the forced decode, a not-parked resume retry, the watched-marker refresh, a queue retry) runs at most once per `notice_probe_s`.
Cards 5 to 8 share one per-producer "probe due" clock for this.

### Decision: episode-state-in-process

`since` and the watched-marker answer live in the producer value across calls of one batten process; only a batten start reads `since` from the status file.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
