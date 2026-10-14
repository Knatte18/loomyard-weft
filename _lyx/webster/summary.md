# Perch budgets and stops: one budget shape, zero-round perches, arbiter only when needed

## Batch 01-reviewbudget-package (done, head 7d81c69ed)

Cards 1–5.
Added `internal/reviewbudget`, the shared review budget key family leaf.
A perch budget of 0 stamps zero rounds on its Bouncer, and the merge-review route switch is gone.
`yamlengine --set` gives a scalar the tag its own text resolves to when the text or the leaf is null.
Loom config moved onto `reviewbudget`, `skip_merge_review` is retired and migrated, and darn's budget keys are settable per run and refuse by layer.
Deviations listed by the digest: `internal/loomcli/wiring_darn_test.go`, `internal/loomengine/{caps,config,config_test}.go`, `internal/loomrecipe/{loomrecipe,reviewbudget_test}.go`, `internal/reviewbudget/reviewbudget_test.go`, `internal/yamlengine/{layers_test,set,set_test}.go`.

## Batch 06-pending-escalation-cause (done, head e2c28a678)

Cards 6–10.
`shedadapters` reads a pending escalation's cause.
New `internal/arbiterstop` holds the one eligibility function: an awaiting stop is eligible only at a hard-cap or circling escalation.
It also reads the facts batten's hold decides on.
Batten holds an eligible stop's notice until the arbiter hands on and measures the report wait from the hold's end.
The LLM driver stencil sends a twice-failed transient to its fork first, lets the verb decide eligibility and relays no notice after a resume.
Deviations listed by the digest: `internal/arbiterstop/{holdreading,holdreading_test}.go`, `internal/battenshed/{stophold,stophold_test}.go`, `internal/loomengine/arbiter.go`.

## Scope guard

Both batches recorded `done` with no warnings; the deviations above are informational only.

## Verify gate fixes

The gate's first attempt failed on `TestDriverStencil_IsRecipeBlind`: `contracts/stencils/shed/shed-template-driver.md:111` named the loom row PR-Gate.
The fixer reworded it to "whatever row it stops at" in commit `32412da94` (`fix:`), and `lyx gate test --tags integration ./cmd/lyx` passed.
