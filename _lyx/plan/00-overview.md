---
format: 5
approved: false
language: go
---

# Plan: shuttle reads Claude Code's turns right

Shuttle takes every valid Stop, and a turn end without output never ends a run (closes #411, #415, #407).
The Stop payload's `background_tasks` becomes the authoritative outstanding signal, with a transcript fallback that also reads absorbed notifications.
A waiting turn end names what it waits on in the trace.
A turn end without every output file holds the run, notifying an autonomous run's parent once per held turn end through the orch notice queue, and Attach waits on held and legacy asking runs instead of respawning beside them.
`OutcomeAsking`, `Spec.AwaitOperator` and every caller branch that handled an asking result retire, treadle's asking triage included.
Senders and stencils carry the fixed message tail, the redraw settle moves into `shuttle.yaml`, and a replay corpus plus `llm`-tier live scenarios pin the readings.

## Card Index

1 — payload-authority — background_tasks is authoritative; the transcript fallback reads absorbed notifications
2 — waiting-trace — log a waiting turn end's outstanding tasks once
3 — redraw-settle-config — submit_redraw_settle_ms in shuttle.yaml
4 — message-tail — the MessageTail constant, appended by lyx shuttle send
5 — hold-turn-end — Wait holds a turn end without output and notifies the parent; AwaitOperator retires
6 — attach-held-runs — Attach waits on legacy asking runs with a live strand
7 — notifier-wiring — loomcli, webstercli and burlercli runners notify the prime's orch
8 — shed-asking-branches — shed producers drop their asking branches
9 — webster-asking-and-shell-note — websterengine drops its asking error; the background-shell note reports the real outcome
10 — treadle-triage-retire — treadle drops its asking triage
11 — retire-outcome-asking — remove OutcomeAsking and update CLI help and docs
12 — tail-in-stencils-and-steer — the orch, driver and AskUserQuestion texts carry the tail and the hold's rules
13 — replay-corpus — the offline replay corpus and an exported clock seam
14 — live-turn-scenarios — llm-tier live scenarios

## Shared Decisions

### Decision: removal-order

- **Decision:** Wait stops returning asking in card 5 while the `OutcomeAsking` constant stays; cards 8–10 remove every caller branch, and card 11 deletes the constant.
- **Rationale:** each card must build on its own, and a constant with callers cannot be deleted before them.
- **Applies to:** cards 5, 8, 9, 10, 11

### Decision: notice-shape

- **Decision:** the hold notice uses `«` and `»` as its fixed delimiter pair, cuts each agent part to 200 runes, names at most five tasks, and quotes `MessageTail`; `hold.go` is its only builder.
- **Rationale:** D5's bounds and the orch stencil (card 12) must describe one shape.
- **Applies to:** cards 5, 12, 13, 14

### Decision: legacy-asking-value

- **Decision:** the string `asking` survives only as an unexported attach-side constant for records an older binary wrote; `RunState.AskingOffset` is read, never written.
- **Rationale:** D6 attaches such records; nothing new writes them.
- **Applies to:** cards 5, 6, 11

## verify:

go build ./...
go test -tags integration ./...
go test -tags tmux ./...
go vet -tags llm ./...
