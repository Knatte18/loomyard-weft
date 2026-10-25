# Card 8 — drift-stencils-tables

**Create:**
- `contracts/stencils/drift/drift-template-baseline.md`
- `contracts/stencils/drift/drift-template-verify.md`
- `contracts/stencils#DriftTemplateBaseline`
- `contracts/stencils#DriftTemplateVerify`
- `internal/driftengine#BaselineInputs`
- `internal/driftengine#BaselineTable`
- `internal/driftengine#VerifyInputs`
- `internal/driftengine#VerifyTable`
- `internal/driftengine#TestSeatTables`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `internal/darnengine#DarnTable`
- `internal/driftengine#Config`

**Intent:**
Add the two agent stencils (Scope) and the seat tables that spawn them, read at call time through `internal/stencilstore` by the seat engine (PATTERN-stencil-ownership), with the parent and edit directives rendered and skills and models on the launch spec (PATTERN-parent-directive, PATTERN-shuttle-provider-seam).

`drift-template-baseline.md` (D3) tells the agent: read the task's subject (the decision record at a told path in loom, or the board entry through `lyx board get` in darn); decide the regression scope, `off` for a task that changes the model on purpose, `partial` naming with `--skip` the tests that will drift totally, `full` for a pure refactoring; record it with `lyx drift scope --scope ... [--skip <test>]...`, which refuses a second call; and write its reasoning and the drift it expects, freeform, to `baseline.md` at its told path.
When the scope record already exists, the prompt's told `scope_note` says so and the agent writes `baseline.md` alone against it.
`drift-template-verify.md` (D6, D7) tells the agent: it is given the compare's log, the scope record, `baseline.md`, the task's subject as amended (`Added after Discussion` entries win), and the baseline's commit; within its budget of `verify_max_runs` runs of `lyx drift compare` and `lyx drift capture --side`, judge which drift is acceptable against the task and `baseline.md`'s expectation, accept it with `lyx drift accept <test> --reason <text>`, find the cause of the rest and fix it, committing each fix; drift traced to a parent commit merged after the baseline's commit is acceptable and the report names the commit; never run the harness any other way; write reasoning, every acceptance and every fix to `verify-report.md`; stop without passing when unsure or past the budget.
Both carry `{{.parent_directive}}` and `{{.edit_directive}}`; neither names a skill or a step the agents' settings deny (PATTERN-refusal-way-forward).
`stencils.go` embeds both under the `drift/` family, `entries` registers `drift-template-baseline` and `drift-template-verify`, and `roleOpeningStencils` maps the roles `drift-baseline` and `drift-verify` to them, so the parent and edit directive scans cover them.
`TestStencils_TopLevelMarkers` gains one row per stencil pinning its exact marker set, which the table builders fill.

`BaselineTable` and `VerifyTable` follow `darnengine.DarnTable`: an empty required input is refused by name; the model is `cfg.BaselineModel` or `cfg.VerifyModel` resolved against the registry and passed through `shuttleengine.NewModelChoice`; one non-interactive lead seat whose only output is `baseline.md` or `verify-report.md` (never `scope.json`, which Go writes and a fresh spawn's output archive must not move); role prefixes from the constants `baselineRole = "drift-baseline"` and `verifyRole = "drift-verify"`; segments `segmentcolor.Plan` and `segmentcolor.Review`; skills `scribe:prose` for the baseline seat and `scribe:prose`, `scribe:code-quality`, `scribe:testing` plus the hub's language skills for the verify seat; the verify table also renders the PATTERN directive (`pattern.RoleImplementer`), since its agent changes code.
The table carries no gate; the `Drift-Verify` producer sets it (card 11-drift-verify-producer).
Each session's timeout is a package constant, the baseline's shorter than the verify's, since D1 declares no timeout key.

`TestSeatTables` (tier0) is a table over both builders: each missing required input refuses by name; the lead's model choice, role prefix, output file, skills and every marker value come out as told; the baseline table told the `decision-record` subject renders the record's path and told `board-entry` renders the board-entry reading, each distinct; an unparsable model is refused.

**ImpactSummary:** The stencil registry and role map gain two entries, which the stencil scans then cover; the marker test gains two rows.
