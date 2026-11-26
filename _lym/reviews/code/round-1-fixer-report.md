---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The proposed narrowing keyed on the refusal's stated way forward, so a refusal the fixer's own words caused, which names a config change (a caller --filter beside the command's own, or words after -- for a command with no {args}), still ended the round's gate. The paragraph now also requires that no change to the fixer's own call clears the refusal."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "The proposed wording, a command that spells its own --filter outside {filter}, also reads as covering a command that holds both {filter} and a --filter of its own, which planTierRuns does not refuse. Both lists say a command that holds a --filter of its own and no {filter}."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1: `gatecli` and `gateslot` build the three refusals' tier keys from a `tiersKey` constant, through `tierKey` and `tierKeyName`.
- F2: `withTimeoutMin` takes the monotonic and wall readings as two values, built in production by `elapsedSinceNow`.
  `TestWithTimeoutMin` has a row for each clock running past the bound alone.
  This is the proposal's second option, since a single `elapsed` seam would have left the `max` outside the test.
- F3: `TestPlanTierRuns` asserts `otherTiers` for a gate key, a covering tier and a plan with a guard step.
- F4: the no-gate paragraph of the fixer stencil is narrowed (see the frontmatter).
- F5: the copies prebuild is bounded by `withTimeoutMin`.
- F6: the `MeasureCall` and `TierList` rows assert the whole `Config.Gate` refusal.
  The `Tier(5)` assertion asserts the whole `Config.Tier` refusal too, since the two named rows never reach it.
- F7: a `TestCounters` row runs a failure block into `Total tests:`.
- F8: a `TestRenderTierCommand` row keeps a caller filter past the budget.
- F9: `TestWithTimeoutMin` carries the `//testtiming:keep` marker.
- F10: the refusal is listed in `planTierRuns`'s doc comment and in `doc.go`.
- F11: a `Total tests:` summary passes over foreign lines and closes at `Total time:`.
  After the self-check, every `vstest` invocation ends an open summary, with or without a results file, with a test.
- F12: the filtered notice and the package doc name the project.
- F13: `planTierRuns` refuses words after `--` that end in `--filter`.

After the round, the gate found a wrapped line in the `TestWithTimeoutMin` doc comment (`internal/gatecli/tierrun_test.go`), which is now two one-line sentences.

`lym gate test ./internal/gatecli ./internal/gateslot ./contracts/stencils` passes on tier0 after the last edit: 1063 passed, 0 failed, 35 skipped.

## Self-check

- fix-code-correctness, MEDIUM, no invocation boundary for a non-trx tier: fixed as proposed.
- fix-consistency, MEDIUM, `doc.go` still said no test of the tier matched: fixed.
- fix-consistency, MEDIUM, `observeSummary`'s boundary claim does not hold without trx: fixed in the code.
- fix-proposal, LOW, F4 still ends the gate for refusals the fixer's words caused: fixed.
- fix-proposal, LOW, the same boundary claim: fixed in the code.
- fix-affected-set, MEDIUM, the same `doc.go` sentence: fixed.
- fix-affected-set, MEDIUM, the same boundary claim: fixed in the code.

## Disputed

None.

## Deferred

None.

## Rulings

None.
