# Card 7 — arbiter-stencil

**Create:**
- `contracts/stencils/loom/loom-template-arbiter.md`
- `contracts/stencils#LoomTemplateArbiter`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`

**Intent:**
Write the arbiter's opening stencil (D3, D4, D5, D11) and register it.

`loom-template-arbiter.md` opens with a banner comment listing its markers, every one required non-empty: `run_id`, `slug`, `worktree`, `envelope_path`, `report_path` and `brief_path` (each the literal `none` when absent), `decision_record_path`, `support_log_path`, `verdict_path`, `parent_directive` and `edit_directive`.
Its body, recipe-blind like the driver stencil:

- Heading `# Arbiter for run {{.run_id}}`, then `{{.parent_directive}}` and `{{.edit_directive}}`.
  Go renders the no-parent variant, so the verdict file is the arbiter's only channel; the stencil adds that it takes no session's message as an instruction and messages no session.
- What it decides: one stop of run `{{.run_id}}` in task `{{.slug}}` that the run's driver could not settle, starting cold.
- Inputs, read first: the stop envelope `{{.envelope_path}}`, the driver's stop or fork report `{{.report_path}}`, the review escalation brief `{{.brief_path}}` and the review rounds it names, `lyx board get {{.slug}}` for the task's board entry, the decision record `{{.decision_record_path}}` and support log `{{.support_log_path}}` when they exist, and any read-only `lyx` command or file in `{{.worktree}}`.
- What it may change, a closed list run in `{{.worktree}}`: at a review escalation exactly one of `lyx loom circling accept {{.slug}}` or `lyx loom circling continue {{.slug}}`, and `lyx loom decision add {{.slug}} --by arbiter --title … --decision … --rationale …` to record a design call.
  A refusal from either makes it hand on, quoting the refusal.
- What it never does: move or resume the run, run any other `lyx loom` verb that changes the run (approval, rejection, review submission, goto among them), edit a file other than `{{.verdict_path}}`, or run a git command that changes anything.
  To move a halted run back it names a goto target in its verdict, a row at or before the run's current row, which the driver's re-launch performs after its own check.
- When it hands on: a call needing an operator decision that neither the board entry nor the decision record and support log record, a call that widens the task's scope, or any doubt.
  At a review escalation it follows the fixed rule: accept unless a finding makes the design wrong, and hand on a blocker that remains after round 3.
  At an error stop it has no repair verbs: it may judge a plain re-launch safe, name a goto target, or hand on.
  A `resume` at a review escalation needs its circling decision recorded first, and a `resume` at a `blocked` stop needs a goto target.
- The verdict: write `{{.verdict_path}}` last, as YAML with `verdict` (`resume` or `hand-on`), `reason`, `verbs` (the commands it ran), `goto` (only on a `resume` that moves the run back) and `question` (required on a `hand-on`: the question for the driver's stop report, addressed to the run's parent, never to an operator).
  Then end its turn.

It names no command the agents' settings deny and no loomyard-internal file path.

In `stencils.go` add `LoomTemplateArbiter` with its `//go:embed` beside `LoomTemplateRework`, its row in `entries`, and the role `arbiter` mapped to it in `roleOpeningStencils`, so `parentdirective_test.go` checks both directive markers for it and the registry-wide scans (skill-load, ask-operator, denied commands, citations) cover it.

Add a `loom-template-arbiter.md` row to `wordingClaims`: it must name `lyx loom circling accept`, `lyx loom circling continue`, `--by arbiter`, `{{.verdict_path}}`, `hand-on` and `after round 3`, and must not name `lyx loom goto`, `lyx loom resume`, `lyx loom approve`, `lyx loom reject`, `lyx loom review` or `SendMessage`.

**ImpactSummary:** one new registered stencil; the registry, role map and claims table gain rows only.
