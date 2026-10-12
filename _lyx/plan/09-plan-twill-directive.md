# Card 9 — plan-twill-directive

**Create:**
- `internal/loomengine#twillPlanFan`
- `internal/loomengine#planTwillSpawnPoint`
- `internal/loomengine#planTwillRejectionRule`
- `internal/loomengine#planTwillSeverityRule`
- `internal/loomengine#twillLenses`
- `internal/loomengine#planTwillDirective`
- `internal/loomengine#TestPlanSpec_Twill`

**Edit:**
- `internal/loomengine#twillBlocks`
- `internal/loomengine#composePlanPrompt` -> `func composePlanPrompt(stencilsDir, specsDir, decisionRecordPath, planDir, overviewPath, patternDirective, frictionDirective, parentDirective, twillDirective string) ([]byte, error)`
- `internal/loomengine#PlanSpec`
- `internal/loomengine#TestPlanSpec`
- `internal/loomengine#TestPlanSpec_Refuses`
- `internal/loomengine#TestCompose_FrictionDirective`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils#wordingClaims`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomengine#PlanReadyMarker`
- `internal/loomengine#PlanProducerTwill`
- `internal/planparser#LensReviewFileName`
- `internal/loomengine#setOffParagraph`
- `internal/loomengine#writeBurlerYAML`
- `internal/loomengine#lensNames`
- `internal/twilldirective#Directive`
- `internal/burlerengine#ResolveFan`

**Intent:**
D2, D3, D5 and D8's spawn side: with `plan_producer: twill`, `PlanSpec` renders the twill directive between Step 2 and Step 3 and sets `ForkSubagents`; with `single` the prompt keeps today's bytes and `ForkSubagents` stays false.
The row, gates, output file, timeout, resume identity and plan-directory rotation stay `single`'s.

Template, `contracts/stencils/loom/loom-template-plan.md`: the blank line between Step 2's last paragraph ("The verbs above are the whole of what `lyx quarry` offers you …") and `## Step 3` becomes `{{.twill_directive}}` on its own line, so an empty value leaves the blank line.
The header comment documents `twill_directive` as an optional marker rendered by `internal/twilldirective` through `internal/loomengine`, sitting on a line that would otherwise be blank.
The template's opening sentence ("no review logic of your own") stays; the plan host's spawn point overrides it under twill.
No write-scope paragraph is rendered (D5).

`composePlanPrompt` takes `twillDirective`, fills `twilldirective.MarkerName` with `setOffParagraph(twillDirective)`, and adds the marker to the optional names.
Its callers, `PlanSpec` and `TestCompose_FrictionDirective`, pass it (the test passes "").

Shared fan resolution (D8: "Discussion's fan-resolution code is shared, parameterised by fan and row name"): `twillLenses(layout, fan, row)` loads `burler.yaml` at the anchor, resolves `fan` with `burlerengine.ResolveFan` and converts the lenses to `twilldirective.Lens`, with today's two errors parameterised: `"%s: burler.yaml unreadable for twill, which needs the %s fan; fix burler.yaml: %w"` and `"%s: %s fan unresolvable; define %s and its lenses in burler.yaml or restore the template's: %w"`, `row` filling the leading `%s`.
`twillBlocks` calls `twillLenses(layout, twillFan, "Discussion-Write")`, so the Discussion errors keep their text.

`planTwillDirective(layout, stencilsDir)` resolves `twillLenses(layout, twillPlanFan, "Plan-Write")` and renders `twilldirective.Directive` with the plan host's values (D5):
- `ReviewedFiles`: "`<planparser.PlanOverview(anchor)>` and every card file its Card Index names";
- `ReadyMarker`: `PlanReadyMarker(layout)`;
- `RubricPath`: `stencilstore.Path(stencilsDir, "loom-rubric-plan-review")`;
- `FormatStep`: "Step 3 and Step 4"; `SelfCheckStep`: "Step 5"; `SelfCheckCommand`: "lyx loom validate-plan";
- `DispositionPlace`: "`<planparser.PlanDir(anchor)>/lens-review.md`", joined with `planparser.LensReviewFileName`;
- `SpawnPoint`, `RejectionRule`, `SeverityRule`: the three constants, whose values are the texts below verbatim (the `Create` heads above carry an empty placeholder value; the constants hold these strings).

`planTwillSpawnPoint` (its lines joined with "\n"):

```text
This lens review is the one review step you run on your own plan: it overrides this prompt's opening line, which says you have no review logic of your own.
Once Step 2's reading is done, run this procedure once, and before you write anything, read the rubric, state the cards you will write in a text reply, one line each with its number, slug and the decisions it carries, and spawn the forks in that same turn.
Write no file under the plan directory before the Agent call.
Add this line to every fork's prompt, after the fork boilerplate and before its lens text, as the one addition to it: "Report defects only, never a passed check. Give each finding's evidence as the `path:line` or glyph read."
```

`planTwillRejectionRule`:

```text
Fix the plan, or reject the finding. A rejection of a `BLOCKING` or `MEDIUM` finding cites the code read to reject it: a `path:line` or glyph opened in this session and what it shows against the finding. Such a finding you cannot refute from code you have read is fixed. Keeping a waiver, a deferral or an unplanned test against such a finding is a rejection and needs the same citation. A `LOW` finding is rejected with a one-line reason.
```

`planTwillSeverityRule`:

```text
a severity (`BLOCKING` when the plan as written fails its verify or ships a wrong behavior, `MEDIUM` when a decision, criterion or branch goes unreached or untested, `LOW` otherwise)
```

`PlanSpec` calls `planTwillDirective` when `cfg.PlanProducer == PlanProducerTwill`, wrapping its error as `loom: PlanSpec: %w`, passes the directive (empty under `single`) to `composePlanPrompt`, and sets `ForkSubagents` to whether the producer is twill.
Fork authorization follows Discussion's: the shuttle Agent hook admits only `subagent_type: "fork"` while `claude_deny_agent_tool` is true, and installs none while it is false.
What this lets through and its bound: the writer may spawn fork subagents; forks run on the writer's model, read-only and without the Agent tool by the stencil's boilerplate, give up after 15 empty waits of up to five minutes each, and the plan gates and the Plan-Review segment still bound what lands in the plan.
`plan.go`'s file header gains the twill paragraph `discussion.go`'s header carries, adapted: the `twill-plan` fan, the ready marker Go removes before every fresh spawn (card 7), `lens-review.md` (card 8), the fork authorization and the bound; it no longer says the producer has no review logic of its own without the twill exception.

Refusals (`PATTERN-refusal-way-forward`): `contracts/specs/refusal-spec.md` gains, beside the two Discussion-Write twill rows, `| Plan-Write: burler.yaml unreadable for twill | plan_producer is twill and burler.yaml cannot be read or parsed at the writer's spawn | correctness halt | fix burler.yaml |` and `| Plan-Write: twill-plan fan unresolvable | plan_producer is twill and twill-plan is defined in neither burler.yaml nor the embedded template, or names an undefined lens | correctness halt | define twill-plan and its lenses in burler.yaml, or restore the template's |`, formatted like their Discussion neighbours.

Tests:
- `TestPlanSpec` asserts `ForkSubagents` is false and pins the `single` bytes: it seeds a second stencils directory whose `loom-template-plan` is the seeded one with the `{{.twill_directive}}` line turned back into a blank line, renders `PlanSpec` over both, and requires equal prompts (acceptance: "With `single`, the Plan-Write prompt is byte-identical to today's and the spec has `ForkSubagents` false").
- New `TestPlanSpec_Twill`, shaped like `TestDiscussionSpec_Twill`, rows "template fan without burler.yaml" (lenses `criteria-coverage`, `row-reachability`, `forced-edits`, `test-economy` in order) and "burler.yaml redefines the fan" (`fans:\n  twill-plan: [test-economy]\n`, only that lens).
  Each row asserts `ForkSubagents` true, and the prompt holds `PlanReadyMarker(layout)`, the plan rubric path, the `lens-review.md` path, "lyx shuttle await", `subagent_type: "fork"`, "lyx loom validate-plan", "Step 3 and Step 4", the spawn point's first line, the per-fork line, the rejection rule, and exactly the line "Return your findings only as your final message, each with a severity (`BLOCKING` when the plan as written fails its verify or ships a wrong behavior, `MEDIUM` when a decision, criterion or branch goes unreached or untested, `LOW` otherwise), the section or decision it concerns, a one-line summary and the evidence."; that "## Lens review by forks" sits after "## Step 2" and before "## Step 3"; and that the write-scope sentence "and you create its parent directory too" is absent.
- `TestPlanSpec_Refuses` gains a `burlerYAML` and a `wantErrNames` field and two twill rows over a temp-dir location with `PlanProducer: PlanProducerTwill`: a `burler.yaml` that does not parse, and one whose `twill-plan` names an undefined lens; each error names `twill-plan` and `burler.yaml` (acceptance: "An unreadable `burler.yaml` and an unresolvable `twill-plan` each fail the Plan-Write spawn with the refusal-spec text").
  `TestDiscussionSpec_Refuses`' two twill rows keep passing unchanged, pinning the shared function's Discussion text.
- `wordingClaims`' `loom-template-plan.md` entry claims `{{.twill_directive}}` in section "## Step 2 — Explore the codebase", because the lens review spawns once Step 2's reading is done and before Step 3 writes a plan file.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/loomengine` by one.

`docs/overview.md`'s Planner producer line adds that `plan_producer: twill` selects the single plan writer that forks itself once per lens of the `twill-plan` fan in `burler.yaml` after its exploration, writes and self-checks the plan, marks it ready, takes the forks' findings in, records them in `lens-review.md` and rejects a `BLOCKING` or `MEDIUM` finding only with a cited code read.

**ImpactSummary:** Plan-Write's prompt gains an optional marker that renders nothing under `single`; under `twill` the spawn renders the directive, resolves the `twill-plan` fan and authorizes forks, and its two new refusals halt the spawn.

**Commit:** `9: Plan-Write twill: render the lens review directive and authorize forks`
