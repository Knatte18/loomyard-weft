---
format: 5
approved: false
language: go
---

# Plan: Burler lens fan and twill fixer in production for Discussion-Review and Plan-Review

Put the Burler lens fan and a twill fixer into production for Discussion-Review and Plan-Review, each selectable per round.
The reviewer's fan prose moves from Go into stencils, four production fans join the embedded `burler.yaml` template, the fixer gains a fork self-check with its own audit, a round's reviewer and fixer fans are recorded in its usage record when its reviewer is spawned, the Bouncer's lens exclusions follow the next round's fan, and `loom.yaml` gains per-round fan and fixer fan lists while `fan_review` is retired.
Every default leaves today's behaviour unchanged; the hub's own `loom.yaml` is set by the orch after landing.

## Card Index

1 — production-fans — add the review-discussion, review-plan, fix-discussion and fix-plan fans and their lenses to the embedded template
2 — reviewer-fan-stencils — move the fork-reading pre-read and the solo and fanned cluster rules into stencils
3 — twill-fixer-engine — a fixer fan spawns a forking fixer with a self-check part, a fork audit and a refusal naming its keys
4 — burler-run-fix-fan — lyx burler run reads fix-fan from its profile and runs the twill fixer standalone
5 — recorded-round-fans — the review half records the round's fan names before the spawn and the fix half reads them
6 — bouncer-per-round-exclusions — the judge asks for exclude_lenses only when the next round runs the judged round's fan
7 — row-fan-lists — the shed rows take per-round fan lists, a fixer fan list and their model keys from Env
8 — loom-per-round-fan-keys — loom.yaml fan keys become per-round lists, the fixer fan keys join and fan_review retires

## Shared Decisions

- A round's names are written only by the review half's pre-spawn write, through `recordRoundFans`, and read by the fix half, a resumed reviewer and the Bouncer; `fans_recorded`, never the `fan` key, says whether they are there.
- No test pins a stencil's or lens's static prose: byte identity of a solo or single round is proven by rendering the stencil with the new marker removed, and a fanned or twill round by the presence of the rendered part and the lens order.

## verify:

```
go build ./... && go test ./contracts/stencils ./contracts/specs ./internal/burlerengine ./internal/burlercli ./internal/shedadapters ./internal/shedrecipe ./internal/loomengine ./internal/loomcli ./internal/configreg ./internal/configcli ./internal/testkit/envkit ./internal/testkit/shedfake ./internal/loomrecipe && go test -tags integration ./contracts/stencils ./contracts/specs ./internal/burlerengine ./internal/burlercli ./internal/shedadapters ./internal/shedrecipe ./internal/loomengine ./internal/loomcli ./internal/configreg ./internal/configcli ./internal/testkit/envkit ./internal/testkit/shedfake ./internal/loomrecipe
```
