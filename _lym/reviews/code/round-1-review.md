---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:1035
    summary: the hasPlan branch that withholds the plan directory from a sampler's expect agent has no test
    fix: in internal/loomcli/wiring_darn_test.go beside TestWire_SamplerRecipe, add a table over the sampler, darn and tapestry recipes that wires each and asserts the Drift deps plan directory is empty for sampler and darn and planparser.PlanDir of the anchor for tapestry.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1226
    summary: the regression-rows list says each row carries drift.enabled and orders Drift-Expect before Webster and Darn-Write only, omitting the sampler
    fix: in docs/overview.md lines 1226 and 1230, state that the sampler's rows also need drift.scope == full, and add "and before Sampler-Session in sampler" to the Drift-Expect ordering sentence.
---

### [MEDIUM] No test pins the sampler's empty plan directory

**Location:** internal/loomcli/wiring.go:1035
**Issue:** The diff changes the condition from `!darnRun` to `hasPlan`, so a sampler run's expect agent is told no plan, and a recipe with a Plan-Write row still is.
No loomcli test reads the plan directory handed to `driftDeps`; `recipekeyed_test.go:61` covers `fixerPlanDir`, a different function.
A regression to the old condition would give a sampler run a plan directory and fail no test.

### [LOW] The regression-rows section of the overview omits the sampler

**Location:** docs/overview.md:1226
**Issue:** The diff added a sampler paragraph near line 1010 but left the regression-rows section saying every row carries the entry gate `drift.enabled`, and ordering `Drift-Expect` only against Webster and Darn-Write.
The sampler's rows carry the added `drift.scope == full` gate and precede `Sampler-Session`, so the section now underdescribes them.

## Holistic check against the fasit

- The sampler recipe gains darn's five Drift rows, each gated by `drift.enabled` and `drift.scope == full`, so with the scope at its default `""` every row is skipped and the run goes as before.
  `TestDriftRows_EntryGate` covers the scope matrix for sampler and shows tapestry and darn ignore the scope.
- The switch is the existing `drift.scope` key.
  The diff adds no config key, and the `template.yaml` change is comment text that matches `baseline.go:143`, which records full for an empty scope, so no reinterpretation is owed.
- The drift verbs resolve any run's worktree from its seed and status, so a sampler worktree is served with only the help and refusal text changed.
- No hunk in `loomrecipe`, `loomcli` or `driftengine` changes tapestry's or darn's stage gates.
  The `fixRoute` change keeps their routes, and `hasPlan` keeps tapestry's plan directory and darn's lack of one.
- The sampler stamp counts a skipped `Drift-Expect` as the predecessor of `Sampler-Session`, and `TestWire_SamplerRecipe` pins it.
- A goto back to `Sampler-Session` after a Drift Stuck raises the entry stamp, so the re-entered session gets a fresh one.
- The diff touches neither `PATTERN.md` nor `pattern/`, the directive declares no change, and no commit carries a `Pattern-Departure:` trailer.
- The claims in `_lym/landing/summary.md` match the diff.

## Rejected

- Code-correctness lens found nothing to reject or keep.
  Its note that Code-Bouncer routes on to Drift-Recheck when `sampler.code_review` is false is harmless and matches darn.
- The code-affected-set note that `amendRows` and `deferHaltRows` omit `Sampler-Session` is rejected.
  A halt after a goto to Sampler-Session cannot arise from a gated stage, and the sampler's drift halts land on Drift-Verify or Drift-Recheck, which both lists admit.
