---
format: 5
approved: false
language: go
---

# Plan: hub orchestrator as a reed strand with automatic handoff

Host the hub orchestrator as an interactive Claude session in a reed strand of the hub's prime worktree, launched and cycled by a new `lyx orch` module.
A detached watcher reads each turn end's context usage from the provider transcript.
When usage crosses a threshold and the session is idle, the watcher has the session write a handoff, clears it, and resumes it from that handoff — never clearing without a written handoff.
Shuttle gains a provider-invariant session surface, claudeengine implements it, and `internal/orchengine` holds config, stencil rendering, persisted state and the cycle state machine behind a seam.
`internal/orchcli` resolves the prime, wires reed and shuttle, and exposes `start`, `status`, `cycle`, `stop` and the hidden `watch` daemon.

## Card Index

1 — shuttle-session-seam — add the optional `SessionCycler` capability and `Runner` methods to read events, read context usage, probe idleness and clear a session
2 — claude-context-usage — read context tokens from the transcript a Stop payload names, degrading to unknown
3 — claude-idle-and-clear — classify an idle input box from a pane capture, and type `/clear`
4 — orch-config — create `orchengine` with its degrading config, registered in `configreg`
5 — orch-stencils — ship the start, handoff and resume stencils and their renders
6 — orch-state — declare the persisted state, told paths, cycle request and fresh-launch reset
7 — orch-start-decisions — decide `start`'s branch and choose its launch prompt
8 — orch-cycle-machine — implement the watcher's per-poll idle check and four-phase cycle with restart rules
9 — orch-watch-loop — wrap the cycle in a single-instance polling loop with a capped error count
10 — orch-cli-skeleton — add `lyx orch` with prime-only resolution and the `status`, `cycle` and `stop` verbs
11 — orch-watch-verb — add the hidden `watch` daemon verb and the production session adapter
12 — orch-start-verb — add the idempotent `start` bootstrap with detached watcher spawn and terminal handover
13 — orch-docs — write the module doc and update the overview, constraints and roadmap
14 — orch-smoke — drive one real cycle in a smoke test and record the open risks

## Shared Decisions

### Decision: session-cycler-capability

The three provider operations the watcher needs — reading context usage, probing an idle session and clearing it — form `shuttleengine.SessionCycler`, an optional capability interface beside `Engine`, rather than new `Engine` methods.
`Engine` has many test fakes across `websterengine`, `webstercli`, `loomcli`, `shuttlecli` and `shuttleengine`, and none of their callers cycles a session; widening `Engine` would force a method set onto every one of them with no behaviour behind it.
`Runner`'s session methods type-assert the capability and return an error naming it when the engine lacks it, the same "cannot, rather than zero" answer `AuditForks` gives for a provider without forks.
Everything Claude-specific stays in `claudeengine` (Shuttle Provider-Seam Invariant).

### Decision: orch-agent-tool

`orchcli` sets `ClaudeDenyAgentTool` false on its own copy of the shuttle config before building its runner.
The orchestrator relies on subagents and forks as the operator's own `claude` session does today, and shuttle's default denies the `Agent` tool in every run.
The override touches only the orch runner's config value; `shuttle.yaml` and every other module keep the deny.

### Decision: orch-session-spec

The orchestrator run's `shuttleengine.Spec` is `Interactive: true`, `AwaitOperator: true`, `Role: "orch"`, `NameOverride: "orch"`, `Display.Focus: true`, with `Model`/`Effort` from the orch config (D3).
Its single `OutputFiles` entry is a fresh `session-<UTC timestamp>.never` path under `.lyx/orch/`, a sentinel nothing ever writes.
The spec needs one output file to validate, and the watcher, not shuttle's `Wait`, owns the session's lifetime, so the run never finishes on an output file.
No caller ever `Wait`s on this run.

### Decision: orch-strand-identity

The orchestrator strand is the strand named `orch` in the prime's reed session, and `State.Strand` records its guid.
`start` looks the strand up by name, since that survives a lost state file.
It refuses when two strands carry the name, and adopts a live `orch` strand whose guid differs from the recorded one, resetting the persisted phase.
The watcher and the other verbs act on `State.Strand`.

### Decision: orch-dir-layout

Everything orch writes lives under the prime's `<anchor>/.lyx/orch/` (D12; Durable-vs-Ephemeral State Invariant), declared once in `internal/orchcli/paths.go`:
`state.json` and `state.json.lock`, `watch.lock` (held for the watcher's life), `start.lock`, `cycle-request`, `watch.log` (the detached watcher's stdout and stderr), `handoffs/` (one timestamped file per cycle, all kept), and the per-launch `session-*.never` sentinels.
`orchengine` is told these paths through `orchengine.Paths` and derives none (Told-Geometry Invariant).

### Decision: events-cursor

Events-file positions are byte offsets from `Runner.ReadEvents`, and each is per run: a fresh launch has a new events file, so `ResetForFreshLaunch` zeroes both stored offsets.
A starting watcher sets its read cursor to the offset that bounds its current phase: `PhaseEventsOffset` in a non-idle phase, `LastInjectionOffset` in `idle`.
It then reads forward from there, so every event it ever evaluates lies past the injection that matters, and no per-event offset is needed.
The newest event's arrival time is when the watcher first read it, held in memory, so a watcher restart restarts the idle grace — the cautious direction.

## verify:

```bash
go build ./...
go test ./internal/shuttleengine/... ./internal/orchengine/... ./internal/orchcli/... ./internal/configreg/... ./internal/configcli/... ./contracts/stencils/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/orchengine/... ./internal/orchcli/... ./internal/configreg/... ./internal/configcli/... ./contracts/stencils/... ./cmd/lyx/...
go vet -tags smoke ./internal/shuttleengine/... ./internal/orchcli/...
```
