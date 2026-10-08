---
format: 5
approved: false
language: go
---

# Plan: lens fans for Discussion-Review and Plan-Review, with per-round usage

Discussion-Review and Plan-Review gain an opt-in burler cluster review: `loom.yaml`'s `discussion_fan` and `plan_fan` name a fan from `burler.yaml`, whose resolution falls back per name to the embedded template, which gains the `discussion` and `plan` fans.
A fanned round's reviewer runs on `fan_review`, the segment's judge narrows the fan from round 2 on by exclusions Go accumulates across the generation, and the judge's facts file shows each lens's findings.
Every review round, solo and fanned, leaves a `round-<N>-usage.yaml` with tokens and wall time per half, forks included, read through a new provider-neutral shuttle capability.
The gate-quiescence guard is rewritten on the ground of shuttle's fork wait, and the Webster-Burler recipe comment is corrected.

## Card Index

1 — burler-fan-fallback — resolve fans and lenses per name over the embedded template
2 — discussion-plan-fans — add the discussion and plan fans and their lenses to the template
3 — burler-exclude-held — fall back to the held exclusions when a round's exclusion would empty the fan
4 — shuttle-session-usage — add the provider-neutral session usage reading and run times to a shuttle Result
5 — claude-session-usage — read a Claude session's tokens, forks included, from its transcripts
6 — burler-half-usage — carry each half's usage and times and the lenses run out of a burler round
7 — round-usage-record — write round-N-usage.yaml for every completed review round
8 — accumulated-exclusions — accumulate eligible exclusions across a generation's rounds
9 — lens-facts — show the judge each lens's findings in a fanned segment's facts file
10 — judge-exclusion-prompt — ask only the judge for exclusions, with the permanence and eligibility rules
11 — env-row-fans — thread a row's fan from Env to its BurlerRound and Bouncer rows
12 — loom-fan-config — add discussion_fan, plan_fan and fan_review to loom.yaml
13 — loom-wiring-fans — wire each segment's fan to both of its rows
14 — gate-quiescence — rewrite the gate-quiescence guard and correct the Webster-Burler comment

## Shared Decisions

### Decision: usage-record-holds-lenses-run

- **Decision:** the round's `round-<N>-usage.yaml` is the one record of which fan and lenses a round ran; exclusion eligibility (card 8) and the facts' lens section (card 9) read it through `readRoundUsage`.
- **Rationale:** D8 already records the fan and lenses per round, so a second record of the same facts could drift from it; a missing record makes the next round's exclusions ineligible, which runs more lenses and never fewer.
- **Applies to:** cards 7, 8, 9

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
