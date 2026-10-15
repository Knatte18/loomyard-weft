---
format: 5
approved: false
language: go
---

# Plan: creel stage 1 — the per-hub message daemon, standalone

Build creel stage 1: a per-hub daemon with a message pool folded from an append-only journal, every verb of the board design (`post`, `inject`, `interrupt`, `forward`, `read`, `inbox`, `log`, `hold`, `release`, hidden `daemon`), sender identity from process ancestry, and delivery into agent panes through shuttle gated on the receiver being idle.
The engine (`internal/creelengine`) is told its geometry and a `Delivery` seam and never imports reed or a concrete shuttle engine; the CLI (`internal/creelcli`) implements the seam over reed's persisted state and shuttle, which learns to drive a live strand with no run record.
No other module sends through creel when this lands.

## Card Index

1 — agentname-program-senders — form and parse program-sender names and EXTERNAL in agentname
2 — selfexec-replaced — detect a replaced binary without exec
3 — proc-parent-pid — read any process's parent pid
4 — reed-status-pane-pid — report each strand's pane pid from reed status
5 — shuttle-recordless-strand — verified send and guarded interrupt on a strand with no run record
6 — creelengine-journal — geometry, the JSONL journal, its fold and retention
7 — creelengine-pool-verbs — the delivery seam, the pool and its verbs with privilege rules
8 — creelengine-identity — resolve the sender from peer ancestry and claims
9 — creelengine-delivery — reminders, injects, holds, pause and the removed-holder sweep
10 — creelengine-daemon — the daemon, socket protocol and Go client
11 — creel-store-pattern — PATTERN-creel-store with its import scan
12 — hubgeom-creel-geometry — build creel's geometry in hubgeom
13 — creelcli-delivery-adapter — implement the Delivery seam over reed and shuttle
14 — creelcli-verbs — the lyx creel subtree, daemon start and sandbox scenario
15 — creelcli-smoke — tmux-tier and llm-tier delivery proofs
16 — creel-docs — the creel design in doc.go and the overview

## Shared Decisions

### Decision: constants-not-config

- **Decision:** retention (7 days), TTL default and cap (10 minutes, 1 hour), hold cap (1 hour), reminder backoff (5 minutes to 1 hour), probe backoff (1 to 8 seconds), sweep period (30 seconds), body cap (64 KiB) and reminder caps (5 messages, 80 runes) are named constants in `internal/creelengine`; there is no `creel.yaml`.
- **Applies to:** cards 6 to 10

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
