# Card 7 — fixer-session-table

**Create:**
- `contracts/stencils/landing/landing-template-fixer.md`
- `plan:contracts/stencils#LandingTemplateFixer` -> `var LandingTemplateFixer []byte`
- `plan:internal/landingshed#FixerMode` -> `type FixerMode string`
- `plan:internal/landingshed#FixerModeMergeIn` -> `const FixerModeMergeIn FixerMode = "merge-in"`
- `plan:internal/landingshed#FixerModeFinalize` -> `const FixerModeFinalize FixerMode = "finalize"`
- `plan:internal/landingshed#FixerInputs` -> `type FixerInputs struct`
- `plan:internal/landingshed#FixerTable` -> `func FixerTable(in FixerInputs, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/landingshed#TestFixerTable` -> `func TestFixerTable(t *testing.T)`

**Extends:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`

**Uses:**
- `internal/landingshed#describeSessionTable`
- `contracts/specs/landing-fixer-report-spec.md`

**Intent:** Build the merge fixer's cold session (D4): its own stencil, read through `internal/stencilstore` at call time, rendering the parent and edit directives (`PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`).
The stencil `landing-template-fixer` opens with `{{.parent_directive}}` and `{{.edit_directive}}`, is embedded as `LandingTemplateFixer`, registered in `entries`, and mapped under the role key `fixer` in `roleOpeningStencils`, so the directive scans cover it.
It tells the session its mode, `{{.mode}}`, and what to read: the decision record `{{.decision_record_path}}`, the merge record `{{.merge_record_path}}`, the plan directory `{{.plan_dir}}` when one exists, and, after a failed gate, the landing failure record `{{.failure_record_path}}` and the gate's log it names.
In `merge-in` mode it reads what the parent brought in since the aligned tip `{{.aligned_tip}}` up to the merged tip `{{.merged_tip}}` against the decision record, fixes what follows from the record and the failing tests, and escalates a conflict between the parent's design and a decision of the record instead of choosing one.
In `finalize` mode it fixes failing tests only.
In both it fixes and commits on the task branch `{{.task_branch}}`, never pushes, leaves the tree clean, and ends by writing the report at `{{.report_path}}` in the format `landing-fixer-report-spec.md` pins, naming that spec rather than restating it (`PATTERN-producer-pointer-rule`).

`FixerInputs` carries the told paths and values: `StencilsDir`, `Mode`, `DecisionRecordPath`, `MergeRecordPath`, `PlanDir`, `FailureRecordPath`, `AlignedTip`, `MergedTip`, `ReportPath`, `TaskBranch`, `ParentBranch`, `Slug` and `LanguageSkills`.
`FixerTable` refuses an empty required field by name and a stencil missing from `StencilsDir`, as `DescribeTable` does; `AlignedTip` is the tip the parent's changes are read from: the merge record's aligned tip or, when it holds none, the entry's `AlignFrom` fork point, so `merge-in` mode always carries one.
`PlanDir` and `FailureRecordPath` are the table's `Optional` values in both modes: `PlanDir` empty for a darn run, `FailureRecordPath` before a failed gate.
`FixerTable` lists `aligned_tip` among the `Optional` values only when `Mode` is `finalize`, which every darn run's fixer is, so seatengine's blank-value check refuses an empty `AlignedTip` in `merge-in` mode by name.
The table has one lead seat: role prefix `fixer`, segment `segmentcolor.Landing` (`PATTERN-spawn-color`), the stencil, the model resolved from `cfg.Fixer`, the report path as its one output, and the language skills as its skills (`PATTERN-role-skills-typed`), and its `Timeout` is `cfg.FixerTimeoutMin` minutes; the caller sets the gate and stretches the timeout over the turns it may spend.

The test builds a table per mode and pins the role prefix, segment, skills, model, output, timeout and marker values, an empty required field naming itself, an empty optional field rendering empty, `AlignedTip` among them in `finalize` mode, a `merge-in` table built with an empty `AlignedTip` being an error naming it, and a missing stencil being an error.

**ImpactSummary:** One registry entry and one role-map line added; the stencil scans gain a stencil to cover, and no existing session changes.
