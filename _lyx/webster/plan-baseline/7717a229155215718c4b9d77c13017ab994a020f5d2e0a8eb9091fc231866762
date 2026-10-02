---
format: 5
approved: true
language: go
---

# Plan: Gate list — several gates per row, each with its own budget and pass_on_cap

A producer row's gate becomes an ordered list of gate entries, each with its own attempt budget and a `pass_on_cap` flag, so one row can carry both a must-pass check and a one-time request.
Card 1 changes `shuttleengine.GateSpec` to the list, evaluates it per arrival with per-entry consecutive-failure counts, and migrates every caller and test fake in the same commit, because the exported shape change breaks them all at once;
`shedrecipe` keeps its old keys for that one commit, mapped onto a one-entry list.
Card 2 replaces the recipe's `gate`/`gate_attempts` keys with a `gates` list, migrates the embedded loom recipe, and documents the key.
No shipped row changes behaviour.

## Card Index

1 — shuttle-gate-list — `GateSpec` becomes an ordered list of entries evaluated per arrival, with every caller migrated
2 — recipe-gates-key — recipe rows declare a `gates` list in place of `gate`/`gate_attempts`, and the loom recipe migrates

## Shared Decisions

- Counts are in memory per shuttle run; nothing is persisted, and `HistoryEntry.GateAttempts` stays one integer fed by the aggregate `GateOutcome.Attempts`.
- `gate-findings.md` and `gateRepromptText` are unchanged, so a single-entry row's re-prompts are byte-identical to today's.
- No `manifest/roadmap.md` change and no new `CONSTRAINTS.md` invariant.
- Told-Geometry, Transient Stop and Refusal Way-Forward hold without a change: a gate closure keeps capturing its own told paths, a gate verdict never carries the transient mark, and the new `shedrecipe` construction errors are wiring guards under each section's grouped row in `contracts/specs/refusal-spec.md`, so no refusal-spec row is added.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
```
