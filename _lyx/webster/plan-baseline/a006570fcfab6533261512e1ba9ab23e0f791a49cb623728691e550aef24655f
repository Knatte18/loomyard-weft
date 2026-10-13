# Card 5 — tackler-stencil

**Create:**
- `contracts/stencils/loom/loom-template-tackler.md`
- `contracts/stencils#LoomTemplateTackler`
- `internal/loomcli#tacklerStencilName`
- `internal/loomcli#tacklerSkills`
- `internal/loomcli#tacklerPromptInput`
- `internal/loomcli#tacklerPrompt`
- `internal/loomcli#tacklerSpec`
- `internal/loomcli/tacklerprompt_test.go`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomengine#TacklerSettings`
- `internal/loomengine#TacklerRole`

**Intent:**
D7's tackler stencil and its launch, read at call time from the told stencils directory (`PATTERN-stencil-ownership`).

`loom-template-tackler.md` is a lyx-owned, recipe-blind opening stencil beside the arbiter's, with a leading comment that names every marker, as `loom-template-arbiter.md`'s does.
Its markers, each required non-empty: `{{.run_id}}`, `{{.slug}}`, `{{.worktree}}`, `{{.envelope_path}}`, `{{.report_path}}`, `{{.trace_copy}}`, `{{.stderr_path}}`, `{{.trace_dir}}`, `{{.driver_log}}`, `{{.verdict_path}}`, `{{.case_brief_path}}`, `{{.parent_directive}}` and `{{.edit_directive}}`.
The body tells the agent:
- it settles one stop of run `{{.run_id}}` the Go driver could not, starting cold, and the verdict file is its only channel;
- read the stop envelope, the Go driver's stop report so far, the trace copy, the captured stderr, the child traces under `{{.trace_dir}}` and the driver log;
- write only `{{.verdict_path}}` and, on `escalate`, `{{.case_brief_path}}`, and edit no other file;
- run no `lyx` verb that changes the run and no git command that changes anything;
- the verdict is YAML with `verdict` (`resume`, `hand-back` or `escalate`), `reason` always, `goto` optionally on `resume` naming only the current row or the row the halting history entry routed from, and `question`, required on `hand-back` and addressed to the run's parent, never to an operator;
- a `resume` at a `blocked` stop needs a `goto`;
- escalate when it cannot name the cause from what it read or when the cause needs code read and judged, and hand back a scope or design choice;
- the case brief has four fixed parts: the question the arbiter must answer, the facts each with a pointer (file and line, log path and line, or trace id), what it ruled out and how, and its hypothesis marked as one;
- write the verdict last, then end the turn.
It names no recipe, no row and no `lyx loom` verb that moves a run.

`stencils.go` gains the `//go:embed loom/loom-template-tackler.md` var `LoomTemplateTackler`, its `entries` row after the arbiter's, and a `"tackler": {"loom-template-tackler"}` row in `roleOpeningStencils`, so `parentdirective_test.go` enforces both directive markers on it.
`wordingClaims` gains a claim row for the stencil: `wantAll` of `{{.verdict_path}}`, `{{.case_brief_path}}`, `hand-back`, `escalate`, `resume` and `hypothesis`, and `wantNone` of `SendMessage`, `lyx loom goto`, `lyx loom resume`, `lyx loom approve`, `lyx loom reject` and `lyx shed goto`.

`internal/loomcli/tacklerprompt.go` mirrors `arbiterprompt.go`:
- `tacklerPrompt` reads the stencil through `stencilstore.Read` and fills `tacklerPromptInput` (`RunID`, `Slug`, `Worktree`, `EnvelopePath`, `ReportPath`, `TraceCopy`, `StderrPath`, `TraceDir`, `DriverLog`, `VerdictPath`, `CaseBriefPath`), `parentdirective.Directive(stencilsDir, "", false)` (the no-parent variant) and `editdirective.Directive(stencilsDir)`, with an empty path filled as `none` through `pathOrNone`.
- `tacklerSpec` sets `Prompt`, `Skills: tacklerSkills` (empty, so no skill is loaded, `PATTERN-role-skills-typed`), `OutputFiles: []string{verdictPath}`, the resolved `Model`, `Effort`, `Version` and `Timeout` from `TacklerSettings`, `Role: loomengine.TacklerRole`, `Segment: segmentcolor.Coordinator` as the arbiter's (`PATTERN-spawn-color`), `Interactive: false`, `ForkSubagents: false` and an empty `Parent`.
  The case brief is not an output file, since only an `escalate` writes it, so `caseBriefPath` only reaches the prompt input.

Tests in `tacklerprompt_test.go`, beside `TestArbiterPrompt`'s shape and over `seededStencils`:
- `TestTacklerPrompt`: every marker value appears, both directives render, an absent report path reads `none`, and no `{{.` remains.
- `TestTacklerSpec`: the spec carries the settings' model triple and timeout, the verdict path as its only output file, the tackler role, the coordinator segment, an empty skill list, `Interactive` false and an empty parent.
`test-budget.yaml`'s `internal/loomcli` row rises by two.

**ImpactSummary:** one new registered stencil and two registry rows in `contracts/stencils`, plus two new composers in `internal/loomcli` that nothing calls until card 7.
