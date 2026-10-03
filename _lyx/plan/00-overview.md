---
format: 5
approved: true
language: go
---

# Plan: review convergence — the judge decides converged, continue or circling from fresh reviews

Review findings gain a required `class`, with `design` the one gating class.
The Bouncer judge stops reviewing artifacts: Go writes a per-round facts file from the reviews and ledgers, and the judge reads it with the latest review and the previous ledger, returning `CONVERGED`, `CONTINUE` or `CIRCLING`.
`CIRCLING` halts the run `awaiting`, and the operator resolves it with `lyx loom circling accept|continue`.
Every Bouncer row runs on a new `judge` model key.
The cards follow the dependency order: the review contract first, then the verdict vocabulary, the facts the judge reads, the circling record, its Bouncer handling and wiring, and the CLI verbs last.

## Card Index

1 — finding-class — every burler review finding carries a required class; design is the gating class
2 — rubric-class-mapping — each loom rubric defines design for its own segment
3 — judge-model-key — Bouncer rows resolve their model from a new judge key
4 — verdict-vocabulary — the judge verdict becomes CONVERGED, CONTINUE or CIRCLING, with legacy words read only at Call entry
5 — round-facts-file — Go renders a per-round facts file from the reviews and ledgers
6 — judge-reads-facts — the judge reads facts, the latest review and the previous ledger, and decides by the written rule
7 — circling-decision-record — the operator's accept or continue on a circling round is a recorded file with its own refusals
8 — bouncer-circling-resolution — the Bouncer acts on a recorded decision and names the verbs and budget in its Awaiting reason
9 — bouncer-env-slug-budget — loom tells every Bouncer row its slug and live bounce budget
10 — circling-verbs — lyx loom circling accept and continue record the decision
11 — retire-classification-draft — remove the completed roadmap entry and the draft design doc

## Shared Decisions

### Decision: legacy-verdict-words-at-entry-only

A legacy `APPROVED` or `BLOCKING` is read as `CONVERGED` or `CONTINUE` only for a verdict already on disk at Call entry (the clear, the replay, and BurlerProducer's may-advance check, all through `recordedVerdict`).
A harvest — a judge this Call spawned or attached — needs a new word; a legacy word there is archived and the round re-judged.

### Decision: facts-file-is-input-not-output

`round-<N>-facts.md` is regenerated before every judge call and is never one of the judge's declared output files, so shuttle's completion check and the attach match on `judgeOutputs` are unchanged.

### Decision: circling-state-in-the-run-directory

The operator's decision is `round-<N>-circling-decision.md` in the Bouncer's run directory, pending when the verb writes it and settled when the Bouncer records it before Approve.
The run directory is committed with the run and archived by the Call-entry clear, so the decision never outlives its generation.

### Decision: budget-read-through-an-env-seam

The Bouncer cannot see Shed's history, so the "continue will block on budget" sentence reads the live count and budget through an optional `Env.SegmentBounces` seam that loom wires over the status file and `Routing.Bounces`.
A nil seam or a read error drops the sentence and never blocks the Awaiting.

### Decision: verb-spelling-in-the-bouncer-reason

The Bouncer's Awaiting Reason spells `lyx loom circling accept|continue` itself.
Every Bouncer row today is a loom review segment, and a told hint string would add a seam with one caller.

## verify:

```bash
go build ./...
go test ./internal/burlerengine/... ./internal/shedadapters/... ./internal/shedrecipe/... ./internal/loomengine/... ./internal/loomrecipe/... ./internal/loomcli/... ./contracts/... ./cmd/lyx/...
go test -tags integration ./internal/burlerengine/... ./internal/shedadapters/... ./internal/shedrecipe/... ./internal/loomengine/... ./internal/loomrecipe/... ./internal/loomcli/... ./contracts/... ./cmd/lyx/...
go vet -tags smoke ./internal/burlerengine/... ./internal/shedadapters/... ./internal/shedrecipe/... ./internal/loomengine/... ./internal/loomrecipe/... ./internal/loomcli/... ./contracts/... ./cmd/lyx/...
```
