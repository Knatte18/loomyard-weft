# Card 3 — The driver stencil shows the one bare step call

**Edit:**
- `contracts/stencils/shed/shed-template-driver.md`
- `cmd/lyx/driverstencil_test.go`

**Uses:**
- `internal/loomcli/driverprompt_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
Decision record D3: the driver stencil gives a driver no reason to wrap its step call.

`shed-template-driver.md`:
- `## How to invoke a step`: show a fenced code block holding exactly `lyx shed step {{.run_id}}`, run as one background job from the session's own cwd, and state that this line is the whole command: no `exec`, no `LYX_TRACE_ID`, no redirect, no temp file, no `cut`, no script file.
  Say the printed envelope is the whole output to read, `trace_file` names the step's trace and `envelope_path` names the file holding the full envelope.
  Keep the background-job, one-step-per-job, wait-for-own-exit and no-`pgrep -f` rules, and the `When the job left no output` fallback through `last_step.envelope_path`.
- `## Drive directory`: replace the `(cd <drive-dir> && lyx ...)` subshell rule with: never change the session's own cwd; `lyx` calls run bare from it, and a command that needs another directory uses `git -C` or its own `(cd … && …)` subshell.
  Keep the sentence that a wrong directory surfaces as the recipe's own refusal text, reported verbatim.
- `## Error envelopes`: the printed error envelope carries `kind`, `transient`, `trace_file` and `envelope_path`; `friction_dir` and `scratch_dir` are read from the file at `envelope_path`, or from the printed envelope itself when it carries no `envelope_path`.
- `## Parking` (the park marker under `scratch_dir`) and `## Repair records and the stop report` (`friction_dir`, `scratch_dir`): say once, in the first section that needs them, that both, and `next_interrupt_policy`, come from the file at `envelope_path`, or from the printed envelope when it carries no `envelope_path`; later sections point back rather than restate it.
The stencil keeps its header comment, every marker, its parent-directive marker and its recipe-blindness, asks no operator (`PATTERN-parent-directive`), and keeps pointing at `contracts/specs/refusal-spec.md` rather than restating it (`PATTERN-producer-pointer-rule`).
One sentence per line.

Bound: the no-`cd` rule is a prose guard, not a structural one.
A bare `lyx shed step <run-id>` from another directory resolves the run from that directory's own seed, so it refuses (missing seed, ownership) or steps a run seeded there under the same run-id; the run-id the stencil names bounds it, and the refusal-verbatim rule reports the refusal case.

`driverstencil_test.go`: add one test, `t.Parallel`, pinning the invocation shape on `stencils.ShedTemplateDriver`: a fenced block whose only line is `lyx shed step {{.run_id}}`; no `(cd <drive-dir> && lyx` form anywhere; and the stencil names `trace_file` and `envelope_path`.
`TestDriverStencil_IsRecipeBlind` and `internal/loomcli/driverprompt_test.go` must keep passing against the rewritten text.

**ImpactSummary:** the driver session's launch prompt changes for every recipe a driver steps; no Go code reads the stencil's prose beyond the two driver-stencil tests.

**Commit:** 3: driver stencil: one bare lyx shed step call, no cwd change
