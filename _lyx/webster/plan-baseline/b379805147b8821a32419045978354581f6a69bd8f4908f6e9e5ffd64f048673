---
format: 5
approved: true
language: go
---

# Plan: per-role-models

Give every loom role that shares its model with another role a `loom.yaml` key of its own: each Bouncer row's judge pass, each Bouncer row's seed pass, and PR-Rework, resolved at run start and threaded to each spawn.
Stop trusting the judge alone on convergence: Go refuses a CONVERGED verdict over an open BLOCKING finding and any judgment whose ledger drops a key of the previous round's ledger, archiving the judgment so the round is re-judged within the bounce budget.
Record the judge and seed passes in the round's usage record beside the review and fix halves.
With every new key empty, every role runs on the model it runs on today.

## Card Index

1 — loom-config-keys — add the eight new loom.yaml keys, their load-time grammar checks, template comments and docs
2 — rework-model-key — PR-Rework spawns on `rework`, which falls back to `plan`, resolved at run start as well
3 — bouncer-seed-judge-split — BouncerConfig carries separate seed and judge model pairs, filled from today's one triple
4 — per-segment-bouncer-threading — resolve each segment's judge and seed at run start, thread them per Bouncer row, retire the row's model keys
5 — convergence-guard — refuse a CONVERGED over an open BLOCKING finding and a ledger that drops a key; archive and re-judge
6 — usage-record-merge — usage record gains seed and judge lists with merge-on-write, and a record without a review half reads as missing
7 — bouncer-usage-entries — the Bouncer records every seed and judge pass whose result reached it

## Shared Decisions

### Decision: bouncer-models-keyed-by-row

- **Decision:** The resolved seed and judge pair reaches a Bouncer row through `Env.RowBouncerModels`, a map keyed by row name, filled by loomcli's wire from `JudgeSettings`' three per-segment fields; a Bouncer row with no entry is a construction error.
- **Rationale:** The decision record's D4 resolves per segment and threads "keyed by segment"; the registry constructor is told a row's name and never its segment, and the review and fix lists it cites as the pattern are threaded by row name in `RowReviewModels`, so row name is the key that reaches the entry without a new seam, and the segment-to-row mapping lives in the one wiring site that already maps segments to rows.
- **Applies to:** cards 3, 4

### Decision: one-pair-type-per-layer

- **Decision:** `shedadapters.BouncerModels` and `loomengine.BouncerModels` are two structurally identical types, `Seed` and `Judge` of `burlerengine.ModelChoice`, converted in loomcli's wire by a type conversion.
- **Rationale:** loomengine does not import shedadapters and should not for one struct; identical shapes keep the conversion field-free.
- **Applies to:** cards 3, 4

### Decision: usage-halves-as-pointers

- **Decision:** `roundUsage.Review` and `roundUsage.Fix` become `*halfUsage` with `omitempty`, so a record's review half is present exactly when its key is, and D11's readers test for a nil review half.
- **Rationale:** D11 needs "no review half" to be decidable, and a zero-valued half is a legal reading of a half on the provider default with no terminal result; the on-disk form of a Burler-written record is unchanged.
- **Applies to:** cards 6, 7

### Decision: replay-guard-skips-an-escalated-round

- **Decision:** On the replay path, the convergence guard runs only when the round carries neither an escalation record nor a circling decision file; a round the parent has been handed, or has decided, stands as recorded.
- **Rationale:** D7 bounds the guard so a pending accept is never refused, and an escalated or decided round's judgment is what the parent's decision settles over; archiving it would leave the decision pointing at nothing.
  The decision file is checked as well as the escalation record because `RecordCirclingDecision` also records a decision over a legacy CIRCLING round that carries no escalation record.
- **Applies to:** card 5

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
