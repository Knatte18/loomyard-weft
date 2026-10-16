# Burler lens fan and twill fixer in production for Discussion-Review and Plan-Review

All four batches reached `done` with no recovery.
No batch reported a `Pattern-Departure:` trailer, and no `warnings` came back.

## Batches

- **01-production-fans** (head `9ea64b16d`): the review-discussion, review-plan, fix-discussion and fix-plan fans and their lenses join the embedded template.
  The fork-reading pre-read and the solo and fanned cluster rules moved into stencils.
  A fixer fan spawns a forking fixer with a self-check part, a fork audit and a refusal naming its keys.
  Deviations: `internal/burlerengine/cluster.go`, `fix.go`, `fix_test.go`, `profile.go`.
- **04-burler-run-fix-fan** (head `38389e184`): `lyx burler run` reads `fix-fan` from its profile and runs the twill fixer standalone.
  The review half records the round's fan names before the spawn, and the fix half, a resumed reviewer and the Bouncer read them.
  The Bouncer asks for `exclude_lenses` only when the next round runs the judged round's fan.
  Deviations: `internal/shedadapters/burlerfix_test.go`, `burlerreview_test.go`.
- **07-row-fan-lists** (head `a90f3430a`): the shed rows take per-round fan lists, a fixer fan list and their model keys from `Env`.
  Deviations: `internal/loomcli/wiring.go`, `internal/shedrecipe/entries_burler.go`.
- **08-loom-per-round-fan-keys** (head `bfa8e8689`): `loom.yaml` fan keys become per-round lists, the fixer fan keys join, and `fan_review` retires with a reconcile line.
  Deviations: `internal/loomcli/wiring.go`, `internal/loomengine/review.go`.

## Scope guard

The glyph scope guard listed the deviations above as files changed outside the batches' own targets.
They are informational and none made a batch stuck.
The plan-level verify gate runs after this turn ends.
