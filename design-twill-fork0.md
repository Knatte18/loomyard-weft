# Twill fork0: blind lens review, strict critical fixer, branch-coverage lens

`twill-fork0` · Task · enhancement, loom, stencil, burler · running · PR-Align

## Goal

Build the Twill design the benches settled for Plan-Write (operator, 2026-10-14): fork0 writes the plan, the master reviews it through blind lens forks and judges their findings as a strict critical fixer, with one new lens for the shared blind spot.

## Evidence

- `docs/research/twill-fixer-bench.md`: the critical fixer turned down no real finding and all its disputes and own fixes were right; the lenses share blind spots no judge can fix.
- `docs/research/twill-fork0-bench.md`: fork0 against after, both with the critical fixer; cost level, fork0 lenses found more real findings (58 against 41), key level, one wrong disposition in 106.
- `docs/research/twill-rationale-bench.md`: a fork0 rationale file changed no outcome, so it is not built; its section "Outcome (operator, 2026-10-14)" states the design.
- The bench's prompt fragments, a working prototype of the flow, are in the hub prime's git-ignored `.scratch/twill-fork0-bench/setup/` (`fork0-preamble.md`, `fork0-block.md`, `critical-judge.md`) and `.scratch/twill-fixer-bench/` for the critical judge's first version.

## Scope

1. **fork0 flow at Plan-Write.**
   The master (the Plan-Write session) explores the code, then spawns one fork, fork0, which writes the plan files and passes `lyx loom validate-plan`.
   The master then reads the plan files, never fork0's reasoning, spawns the lens forks from its own session, so the lenses hold the exploration and the plan but not the writer's reasoning, and judges and fixes their findings itself.
2. **Strict critical fixer as the judge step** (operator: the strict prompt is essential).
   Each finding is weighed twice, its premise against the code and the decision record, then the change it proposes, and gets one of three dispositions with cited evidence: dispute (premise false, or every fix worse), fix with its own change, or fix as proposed; modelled on `contracts/stencils/burler/burler-step-fix.md`.
   It replaces today's "fix or reject with a one-line reason" judge step.
3. **New lens `branch-coverage`** in the `twill-plan` fan (`internal/burlerengine/template.yaml`): list every branch and exit the cards' Intents introduce, whether or not the decision record names it (error returns, early returns, the end of a loop or scan without a match), and find the planned row that reaches each; a branch no row reaches is a finding.
   This is the blind spot every bench arm shared (KI2, KI4, KW4, KP3 in the reports).
4. **`forced-edits` lens:** one added sentence, a new type, function or executor that must be registered in a table or list (such as a guard table) is a forced edit; KW3 in the reports.
5. **Wrong parent directory** (from `twill-plan-followups`): a miss every `twill-plan` bench arm shared, a check against the wrong parent directory; cover it with a sentence in the fitting lens or the plan rubric (`docs/research/twill-plan-bench.md`).

Discussion decides whether the critical judge step (2) also applies where twill reviews Discussion-Write: the benches measured Plan-Write only.

## Not built

- The fork0 rationale file (benched, no gain).
- Fewer Plan-Review rounds with twill: measured by `convergence-plan`, not here.
- An efficiency audit of the twill planner, wanted later (operator, 2026-10-12): the plan writer's long session, not the forks or `validate-plan` failures, dominates cost (`docs/research/token-usage-by-role.md`).
- Untested ideas from the bench agent: merge two lenses (such as `test-economy` into `row-reachability`), have forks batch their reads into one or two calls, and cut the writer's `validate-plan` rounds.
