# Lens fans for Discussion-Review and Plan-Review, with per-round usage

All three batches recorded `done`, head `0a3a3846949cc62f4c3f02d15fd342a277440d98`.

## Batch 01 — burler-fan-fallback (cards 1–6)

Fans and lenses resolve per name over the embedded template.
The template gained the discussion and plan fans and their lenses.
A round whose exclusion would empty the fan falls back to the held exclusions.
Shuttle's Result gained a provider-neutral session usage reading and run times.
The Claude engine reads session tokens, forks included, from transcripts.
Each burler half carries its usage, times and the lenses run.
Deviations: `cmd/lyx/testdata/test-budget.yaml` (shuttleengine budget) and `internal/shuttleengine/waitshadow_test.go` (zero `EndedAt` before comparing).

## Batch 07 — round-usage-record (cards 7–11)

Every completed review round writes `round-N-usage.yaml`.
Eligible exclusions accumulate across a generation's rounds.
A fanned segment's facts file shows each lens's findings.
Only the judge is asked for exclusions, under the permanence and eligibility rules.
A row's fan threads from Env to its BurlerRound and Bouncer rows.
A missing usage record makes exclusions ineligible, so the full fan runs.
A fanned round 1 drops focus-file excludes and logs a warning.
Deviations: `cmd/lyx/testdata/test-budget.yaml`, `internal/testkit/envkit/envkit.go`, `internal/shedadapters/{bouncer,bouncerprompt,focus,bouncerfacts}.go`, `bouncerprompt_test.go`, `internal/shedrecipe/{recipe.go,entries_bouncer_test.go,entries_burler_test.go}`.

## Batch 12 — loom-fan-config (cards 12–14)

`loom.yaml` gained `discussion_fan`, `plan_fan` and `fan_review`, and each segment's fan is wired to both of its rows.
The gate-quiescence guard was rewritten and the Webster-Burler comment corrected.
No deviations.

The plan-level verify was not run by the forks; the gate runs it.
