---
format: 5
approved: true
language: go
---

# Plan: orch-cycle-hardening

`lyx orch` cycles its context on a fresh reading, by `/compact` or `/clear` as `orch.yaml`'s `cycle_mode` says, and the hub runs under it for days without hand repair.
A compaction boundary becomes a context reading read from the transcript's end, the watcher re-reads before firing, compact becomes the default cycle, `permission_mode` narrows to `bypass`, and batten tells the orch about run changes through a notice queue that only the orch watcher types from, so the orch keeps no watcher of its own.
The hardening items follow: a reset reading on adopt, PID reuse at adopt, a named hold for a too-short pane, and smoke coverage.
One obligation sits outside the cards: at landing, the `orch-owns-the-loop` board entry's "A watch command" section is amended, since cards 7 to 9 cover it.

## Card Index

1 — context-reading — make a compaction boundary a reading, read the transcript backwards
2 — reread-and-reset — re-read before an automatic cycle fires, reset the reading on fresh launch
3 — orch-config-modes — add cycle_mode and refuse any permission_mode but bypass
4 — compact-seam — add the provider's compact command to the shuttle seam
5 — compact-cycle — run the compact cycle in the watcher behind cycle_mode
6 — pane-too-short — record a too-short orch pane as the reason cycles are held
7 — notice-queue — add the orch notice queue and its idle-gated delivery, plus the single-writer PATTERN entry
8 — batten-notices — write one notice per episode from the Run-Shed row
9 — batten-notice-wiring — wire batten's notice seams to the prime's orch queue
10 — stencils-no-watcher — drop watcher re-arming from the stencils and teach the notice rule
11 — adopt-pid-reuse — let an adopt through when the holder's pid is proven reused
12 — orch-smoke — smoke-test the compact cycle, the bypass footer and a large-session adopt

## Shared Decisions

### Decision: provider-neutral seam types

- **Decision:** the context reading and the idle probe cross the shuttle seam as provider-neutral structs declared in `internal/shuttleengine` (`ContextReading`, `IdleProbe`); the transcript shape, the pane geometry and the `/compact` text stay in `internal/shuttleengine/claudeengine`.
- **Rationale:** PATTERN-shuttle-provider-seam.
- **Applies to:** cards 1, 4, 5, 6

### Decision: cycle mode default and test split

- **Decision:** an absent `cycle_mode` is compact; the existing four-phase watcher tests pin `cycle_mode: clear` in their shared env and stay otherwise unchanged, and compact-mode tests live in their own file.
- **Rationale:** acceptance requires the clear-mode tests to pass unchanged apart from stencil text.
- **Applies to:** cards 3, 5, 6, 12

### Decision: notices cross modules through seams

- **Decision:** `battenshed` renders notices and tracks episodes but queues them only through an injected `Notify` seam; `battencli` wires it to `orchengine.QueueNotice` with paths from `orchcli.PrimePaths`.
- **Rationale:** `battenshed`'s import allowlist and PATTERN-told-geometry; the queue path is declared once in `internal/orchcli/paths.go`.
- **Applies to:** cards 7, 8, 9

## verify:

go build ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./internal/orchcli/ ./internal/orchengine/ ./internal/shuttleengine/... ./internal/battencli/
