# Card 8 — rework-stencil-spec

**Edit:**
- `contracts/stencils/stencils.go`

**Create:**
- `contracts/stencils/loom/loom-template-rework.md`
- `internal/loomengine/rework.go`
- `internal/loomengine/rework_test.go`

**Uses:**
- `internal/loomengine/plan.go`
- `contracts/stencils/loom/loom-template-plan.md`

**Intent:**
Add the rework agent's prompt and its Spec factory (D5).
`loom-template-rework.md` instructs one autonomous session to turn the pending rejection's findings into new cards appended to the plan.
Its markers: `rejection_path` (the record; the findings are its `findings` field), `plan_dir`, `overview_path`, `decision_record_path`, `coverage_path`, `plan_stencil_path`, `specs_dir`, plus the optional `pattern_directive` and friction markers.
It states, and does not restate the card format:
- the card format, glyph lookup rules and `validate-plan` self-check are exactly those of the plan stencil at `plan_stencil_path`, which the agent reads first (Producer Pointer-Rule Invariant);
- read the findings, the plan, the decision record, and the task diff with read-only git;
- append one or more cards numbered after the highest card committed at HEAD, plus their Card Index lines, and change nothing else: no existing card, no frontmatter key (the `approved:` flag included), no plan-level section;
  Go checks this after the session against the plan committed at HEAD, and a card file absent at HEAD is a leftover of an interrupted attempt of this same round that the agent owns;
- every finding is covered by at least one new card, and several findings may share one;
- write the coverage file LAST, mapping each finding to its card numbers, as the completion signal.
Register it in `stencils.go` as `LoomTemplateRework` with an `entries` row beside `loom-template-plan`.
`rework.go` adds `ReworkSpec(layout *lyxcwd.Location, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry) (shuttleengine.Spec, error)` and its `composeReworkPrompt`, mirroring `PlanSpec`:
it reuses the plan role's model-spec (`cfg.Plan`) and `cfg.PlanTimeoutMin`, because rework is planning and adding config keys buys nothing;
`OutputFiles` is `LoomReworkCoveragePath(layout)`, `Role` is `"rework"`, `Interactive` is false, the friction note name is `PR-Rework`, and `plan_stencil_path` is `stencilstore.Path(stencilsDir, "loom-template-plan")`.
`rework_test.go` seeds a stencils directory, asserts every marker renders its told path, the output file is the coverage path, and a missing stencil fails composition.

**ImpactSummary:** Adds one registry row; contracts/stencils' registry test gains a file-entry pair and nothing else changes.
