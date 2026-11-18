---
dispositions:
  - id: F12
    disposition: fixed
---

## Changes

- F12 (fixed): in `_lym/plan/13-gate-mutate.md` Tests, the `mutation_budget_min` row of `TestGateMutate` now says that the coverage build's bound and the budget share the one `minuteUnit`, so only the config values separate them.
  The row passes a `minuteUnit` of one millisecond, and its hub config sets `mutation_budget_min: 1` and `redundancy_build_timeout_min: 100000`.
  That gives a one-millisecond budget and a 100-second build bound, so the build finishes and the mutants are planned before the spent budget leaves them unjudged.
  Premise checked: `internal/gatecli/redundant.go:255` and `internal/gatecli/redundancy.go:462` scale the build bound by `minuteUnit`, and `internal/gateslot/template.yaml:5` defaults the bound to 10.
  The one-millisecond unit came from the self-check below; it goes beyond the review's proposed text, which left the unit open.

## Self-check

- fix-proposal, MEDIUM: the 100000 bound is enough only when `minuteUnit` is about a millisecond or longer, and the row never pinned the unit. Fixed as proposed: the row now pins `minuteUnit` to one millisecond and states the resulting budget and bound.
- fix-claims, LOW: the same finding as fix-proposal's, merged with it and fixed by the same change.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
