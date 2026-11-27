---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The code, doc and test changes are made as proposed. The proposed update of the change description is not made: it lives at _lym/landing/summary.md, and _lym is owned by the loop, outside this round's write surface."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer report

## Changes

- **F1.** `scanGuards` leaves a scope package's guard test to the guard step unless a listed tier is or covers its tier.
  `computeScope` stores those tiers on `scope.listedTiers` through the new `coveredTiers`.
  The guard-step section of `internal/gatecli/doc.go`, the `--guards` sentence of the help text and the "no guard test" log line follow.
  A `TestGateTest_Scopes` row pins a dependent package's guards under `--card` with and without `--tier 1`.
- **F2.** `cardDependents` takes the stderr writer and names the narrowing, the module and the cause there.
  The main-package listing of `--main-packages` now runs before the widening,
  because the new line otherwise printed ahead of that flag's refusal and failed its existing row.
- **F3.** The implementer stencil sanctions the fix for a `//lym:guard` test failing in a listed tier's run or the guard step.
- **F4.** The vet-step section of `internal/gatecli/doc.go` speaks of the card scope's packages.
- **F5.** The `--map --card` test adds a package importing `solo` and expects both maps built.
- **F6.** The success row is gone.
  The one remaining case is now `TestCardDependents_UnlistableModuleKeepsTheCardsPackagesAndSaysSo`.
  The commit carries a `Loosened-Check:` line.
- **F7.** `dependentsFor` builds its result from the cleaned spelling.

F1, F2, F4 and F6 share files and land in one commit.
F3, F5 and F7 each have their own, and the self-check fixes are one commit on top.

Gates after the last change: `lym gate test --tier 0` over `internal/gatecli`, `internal/impactset`, `contracts/stencils` and `cmd/lym` passed.
`--tier 1` over the first three passed.
`--tier 1` over `cmd/lym` passed before the self-check fixes, which touch nothing in it.
`lym loom lint-comments` passed after the last change, and `lym loom lint-pattern` passed before the comment fixes.

## Gate finding: tier1 timeout

The round's verify run failed because tier1 exceeded its 30-minute `timeout_min` and was killed.
No code or test was changed for it: the cause was the host hibernating during the run.

- The system journal shows hibernation entry on 2026-11-26 at 22:44:40, about four minutes after tier1 started,
  and the return on 2026-11-27 at 07:20:20.
- The run was killed on wake, since the tier's bound counts wall-clock time.
  A search of its log for failed tests and test-timeout panics found none.
- `lym gate test --gate code-burler-fix`, rerun on the same HEAD, passed:
  tier1 discovered 6822, passed 4574, failed 0, skipped 2248, in 482.1s.
  tier0, tier2 and the tier0 guard step were carried as already green.

The hub's tier1 `timeout_min` needs no change on this evidence.

## Gate finding: reach report

No test was added and nothing was committed for it.

- The report says no mapped test reaches `internal/gatecli.computeScope` or `internal/gatecli.coveredTiers`.
  `TestGateTest_Scopes` drives both through `lym gate test` in a spawned process, which the map run does not run.
- For `coveredTiers`, the row "--card runs a dependent package's guard in the guard step unless a listed tier is or covers its tier"
  asserts the guard step's invocations under the default tier0 and under `--tier 1`, which covers tier0.
  The row passed in the self-check's tier1 run.
- An in-process test of either function would assert the same behaviour a second time.
- The other lines of the report name a reaching test or describe the maps' age and limits.

## Self-check

- fix-consistency, MEDIUM, help text false for computed and `--base` scopes: fixed.
- fix-consistency, MEDIUM, `_lym/landing/summary.md` stale: deferred, outside the write surface.
- fix-code-correctness, MEDIUM, help text false for computed and `--base` scopes: fixed (same change).
- fix-code-correctness, LOW, the "covers" half of the new row untested: fixed.
- fix-affected-set, LOW, doc reads as if only `--card` could drop a guard: fixed, stated as a bound in `internal/gatecli/doc.go`.
- fix-affected-set, LOW, `_lym/landing/summary.md` stale: deferred, outside the write surface.
- fix-proposal, MEDIUM, help text false for computed and `--base` scopes: fixed (same change).
- fix-proposal, LOW, `_lym/landing/summary.md` names the old test: deferred, outside the write surface.

## Disputed

## Deferred with reason

- **`_lym/landing/summary.md` is stale** (the rest of F1's `fix`, and the self-check findings above). It is under `_lym`.
  It needs three changes:
  - The guard step also runs a scope package's guard test whose tier no listed tier is or covers.
  - A failed dependents listing is named on stderr as well as logged.
  - Under Tests, `TestCardDependents` is now `TestCardDependents_UnlistableModuleKeepsTheCardsPackagesAndSaysSo` and covers the failure path only.
- **Not part of any finding, for the parent:** a computed or `--base` scope still drops every guard of an impacted package whatever its tier.
  So a tier0 diff gate over an impacted `cmd/lym` runs its tier1 guards nowhere.
  This predates the change and conflicts with `PATTERN-test-speed`'s guard-step rule.
  It is now stated as a bound in `internal/gatecli/doc.go` and not fixed.

## Rulings
