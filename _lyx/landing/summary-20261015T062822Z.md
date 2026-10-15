# Twill: fork0 plan writer, critical-fixer judge, branch-coverage lens

Plan-Write with lens forks on now runs the fork0 flow the benches settled on (`docs/research/twill-rationale-bench.md`, "Outcome").
The Plan-Write session explores, then spawns one fork, fork0, which writes the plan and passes `lyx loom validate-plan`.
The session reads the finished plan without fork0's reasoning and spawns the lens forks, so the lenses hold the exploration and the plan but not the writer's reasoning.
It then judges their findings as a strict critical fixer.
At level cost, fork0's lenses found more real findings than the old writer-then-lenses flow, with no loss of precision.

## What changes

- **fork0 at Plan-Write.**
  The flow replaces the old after-flow outright; no key or flag selects the old one.
  The procedure, its failure paths and fork0's prompt are host-neutral stencil text in the new `contracts/stencils/twill/twill-writer-fork.md`, rendered by `internal/twilldirective` as an optional part that only a host asking for a writer fork fills.
  Discussion-Write asks for none, so its rendered directive carries no fork0 text.
- **Critical-fixer judge step, at both hosts.**
  The directive's judge step now gives each finding one of three dispositions: dispute (`premise false` or `every fix worse`), fix with your own change, or fix as proposed.
  A dispute needs cited evidence opened in the session; the lens-review record carries one line per finding with its lens, severity and disposition.
  The text follows `contracts/stencils/burler/burler-step-fix.md`, without Burler's write-surface deferral.
  The host's governing input and its dispute evidence kinds arrive as host values: the decision record and code-level kinds at Plan-Write, the board entry and design-level kinds (no `path:line` required) at Discussion-Write.
  The `rejection_rule` host value and Plan-Write's rejection-rule constant are retired.
- **Failure paths.**
  A new `twill-relaunch.md` part renders the paths for a respawned or resumed master in place of the two Agent-call relaunch paragraphs, which fork0 makes ambiguous.
  A re-prompted master (gate re-prompt or parent-review reject) spawns no fork0, edits the plan itself, and runs the lens review once only if it has not yet made its lens spawn call.
  The re-prompt rule replaces the sentence that a re-prompt never spawns forks again, at both hosts.
  The Discussion-Write directive keeps its two relaunch paragraphs.
- **`twill-plan` fan** (`internal/burlerengine/template.yaml`).
  New `branch-coverage` lens after `criteria-coverage`: every branch and exit the cards' Intents introduce, named by the record or not, must be reached by a planned test row.
  `forced-edits` gains a sentence making a registration in a table or list, such as a guard table, a forced edit.
  `criteria-coverage` gains a sentence checking that each path, comparison or default uses the operand the record names, not a near one such as the wrong parent directory.
- Docs: the `twilldirective` package doc, `docs/overview.md`'s twill lines and the `plan_lenses` comment in `internal/loomengine/template.yaml` describe the fork0 flow and the critical judge.

## Design decisions to know

- With lens forks off, the rendered prompt is unchanged.
- fork0 is bounded to one spawn per master session, as the lens forks are.
  A fork0 that returns without a complete plan is finished by the master, not respawned.
- The Plan-Write gate is unchanged: a run with lens forks on still requires a recorded `lens-review.md`, and a missing one fails `lens-review-missing`.
- The twill stencils stay host-neutral under `hostNeutralClaims`; the claims and markers tests (`contracts/stencils/claims_test.go`, `markers_test.go`) are extended, not multiplied.
- The plan rubric is unchanged; the wrong-operand check lives in `criteria-coverage` so it does not land in every Plan-Review round.
- Not built: fork0 at Discussion-Write, a fork0 rationale file, and any mechanical check on the lens review's content.

## To check by hand

- The critical-fixer judge now applies at Discussion-Write, though the benches measured Plan-Write only.
  Check whether its design-level evidence kinds, bounded by the writer's Step 2 ban on exact signatures and `file:line` citations, let the judge refute a finding as often as the code-level kinds do at Plan-Write.
- A hub `burler.yaml` that overrides the `twill-plan` fan keeps its own lens list and misses `branch-coverage` until edited; the prime hub overrides no twill fan today.
- No end-to-end Plan-Write run with a live fork0 was made here, and none of the resume paths (master resumed in flight, before or after fork0, at or after the lens spawn) ran against a real reed resume; only the rendered prompts and the gate are pinned by tests.
