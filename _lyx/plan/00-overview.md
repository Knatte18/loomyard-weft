---
format: 5
approved: true
language: go
---

# Plan: judge-cap-call

At a spent review budget the Bouncer settles the round itself from the judge's per-finding fix tags instead of escalating to the parent and its arbiter.
It escalates only where a run is stuck (a design-wrong finding open at a new per-perch hard cap, or a circling design-wrong finding from the soft cap on) or where the rule cannot read its inputs.
The cards build bottom-up: the judge's tag, the `hard-cap` cause, the Bouncer's hard cap, its Env wiring, the soft-cap rule, then the loom and darn config keys, the `circling continue` refusal and the arbiter and orch stencils.

## Card Index

1 — ledger-fix-tag — the judge tags every open ledger entry mechanical or design-wrong, and the ledger parser keeps the tag
2 — hard-cap-cause — hard-cap becomes a third cause in the escalation record and the circling decision file
3 — bouncer-hard-cap — BouncerConfig.HardCap: a recorded continue at the cap re-escalates hard-cap, escalations there offer only accept, continues past the soft cap are exempt
4 — env-hard-cap — shedrecipe.Env.RowHardMaxBounces reaches each Bouncer row's HardCap
5 — soft-cap-rule — the Bouncer settles a spent budget by the soft-cap rule, auto-accept's replay and blocking paths included
6 — loom-hard-cap-config — loom.yaml hard-cap keys, their load refusals with per-layer routes, and their wiring into the loom Bouncer rows
7 — darn-hard-cap-config — darn.yaml's hard cap for the Darn-Review perch, its load refusals and wiring
8 — circling-continue-hard-cap — lyx loom circling continue refuses at or past the perch's hard cap
9 — arbiter-orch-stencils — the arbiter's review rule per escalation cause, and the orch's escalation guidance and small-task advice

## Shared Decisions

- The hard cap is a round number in the segment's current generation, never Shed's bounce count, so a goto that resets the count cannot pass it.
- `BouncerConfig.HardCap` and `shedrecipe.Env.RowHardMaxBounces` treat 0 as no hard cap, so every Bouncer built without the wiring behaves as before; loom and darn always tell a resolved cap of at least 1.
- A ledger entry counts as design-wrong when it is open, BLOCKING or MEDIUM, and not tagged `mechanical`.
- The escalation brief's decide options live in two stencil fragments Go picks by the hard cap, so Go interpolates values only.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
