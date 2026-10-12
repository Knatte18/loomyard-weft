# Card 8 — arbiter-prompt-spec

**Create:**
- `internal/loomcli/arbiterprompt.go`
- `internal/loomcli/arbiterprompt_test.go`
- `plan:internal/loomcli#arbiterPromptInput` -> `type arbiterPromptInput struct`
- `plan:internal/loomcli#arbiterPrompt` -> `func arbiterPrompt(stencilsDir string, in arbiterPromptInput) (string, error)`
- `plan:internal/loomcli#arbiterSpec` -> `func arbiterSpec(prompt, verdictPath string, settings loomengine.ArbiterSettings) shuttleengine.Spec`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:contracts/stencils#LoomTemplateArbiter`
- `plan:internal/loomengine#ArbiterSettings`
- `plan:internal/loomengine#ArbiterRole`

**Intent:**
Compose the arbiter's launch prompt and shuttle spec, pure composers in a new file beside `driverprompt.go` and `driverspec.go`.

`arbiterPromptInput` carries the run id, slug, worktree, envelope path, report path, brief path, decision record path, support log path and verdict path.
`arbiterPrompt` reads `loom-template-arbiter` through `stencilstore.Read` at call time (`PATTERN-stencil-ownership`), renders `parentdirective.Directive(stencilsDir, "", false)` (the no-parent variant, D11) and `editdirective.Directive(stencilsDir)`, fills every marker with `stencil.Fill`, and fills an empty report or brief path with `none`.
It returns the read or fill error wrapped with the stencil's name.

`arbiterSpec` pins, each with its reason in the doc comment as `driverSpec` does: `Prompt`; `OutputFiles` holding the verdict path alone, so shuttle's completion check reads the verdict through `allOutputFilesExist` (`PATTERN-completion-signal`) and reed drops a finished strand on resume; `Model`, `Effort` and `Version` from the settings; `Timeout` the settings' timeout; `Role` `loomengine.ArbiterRole`, which reed forms into `<shortname>:<slug>:arbiter` through `internal/agentname`, with no `NameOverride` so a second attempt numbers its role; `Skills` an empty list (`PATTERN-role-skills-typed`); `Segment` `segmentcolor.Coordinator`, the driver's (`PATTERN-spawn-color`); `Interactive` false; `ForkSubagents` false; `Parent` empty; `Display` at its zero value.

`arbiterprompt_test.go` holds `TestArbiterPrompt`, which fills the shipped stencil from a temp stencils dir and asserts every marker value appears, `none` stands for an absent report and brief, the no-parent directive and the edit directive are rendered, and no `{{` survives, and `TestArbiterSpec`, which asserts each pinned field.
Raise `internal/loomcli`'s row in the test budget by the tests added.

**ImpactSummary:** two new unexported composers with no caller until card 10; only the test budget row changes.
