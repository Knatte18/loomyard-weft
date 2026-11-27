```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
- `agentname.RoleFromText` lowers the text and joins its words with dashes; refused characters stay → `TestRoleFromText` (`internal/agentname/agentname_test.go`). Pinned.
- `agentname.RoleRule` text → pinned only as static prose. Exempt.
- `roleFromFlag` normalizes the value and refuses it with the rule first (`internal/shuttlecli/agent.go:115-122`) → **none**.
- `lym shuttle open --role` passes the normalized role to `OpenSpec` and refuses an invalid one (`internal/shuttlecli/open.go:81-87`) → **none**. `TestCommand_Run_FlagValidation` covers only `--role is required` and a bad `--model`. `TestOpenCmd_ModelAliasEngine` passes only roles that are already valid.
- `lym shuttle agent` verb: `--name` refusal, `--model` refusal, the geometry-to-Inputs mapping (Slug, ParentName, HubShortname, HubPath, WorktreeRoot, language skills), the `StartSession` call and the output envelope → **none**.
- `operatoragent.SessionSpec` sets Interactive, Unawaited, Role, ConfigRole, ModelChoice, PromptEngine, Display and Skills → `TestSessionSpec` (`internal/operatoragent/spec_test.go`). Pinned.
- `RenderPrompt` names the hub, worktree and place (board entry or prime) → `TestSessionSpec`'s `TaskWorktreeNamesItsEntry` and `PrimeNamesThePrime` rows. Pinned.
- `RenderPrompt` refuses an empty required input by name → `TestSessionSpec/RefusesAnEmptyRequiredInput`. Pinned.
- `RenderPrompt` renders the edit directive for the engine → `stencilkit.RequireEditDirectiveOf` in `TestSessionSpec`. Pinned.
- `RenderPrompt` renders the parent directive from ParentName (parent variant with a parent, no-parent variant in the prime) → **none at render level**. `TestStencils_OpeningStencilsRenderParentDirective` pins only that the stencil carries the marker.
- `RenderPrompt` fills the pattern directive from WorktreeRoot when the worktree has a PATTERN overview → **none**. Both rows use non-existent paths, so only the empty branch runs.
- `operatoragent-template-start` registration and seeding → indirectly by `stencilkit.Seed` plus `ReadForEngine` in `TestSessionSpec`. Pinned.
- `roleOpeningStencils["agent"]` → `TestStencils_OpeningStencilsRenderParentDirective` and `TestStencils_OpeningStencilsRenderEditDirective` (`contracts/stencils/parentdirective_test.go:124,142`). Pinned.
- `agentconfig.RoleAgent` in `roles` with a template list → `TestLoad_TemplateListsEveryRole` (`internal/agentconfig/config_test.go:70`). Pinned.
- Button rows (new run line and ask texts in task, beam and prime) → `TestLoadConfig_TemplateDefaultsResolve` (`internal/reedengine/config_test.go:99-121`). Pinned.
- The parity producer row for `internal/operatoragent#SessionSpec` → `TestProducers_CodexManifestHoldsClaudes`, through the producer table. Pinned.
- Test economy: no duplicate test, no per-helper test and no tautology found. The `ValidateRole` assertion inside `TestRoleFromText` pins that the output has the grammar's shape, which `TestValidationRejections` does not cover. Not redundant.

**Findings:**

1. **MEDIUM, design.** `internal/shuttlecli/agent.go:115` (`roleFromFlag`) and `internal/shuttlecli/open.go:81`. The `--role` / `--name` normalization and the rule-first refusal message have no test. This is the board entry's first ask: name the rule on a refusal, or turn spaces into dashes. Evidence: there are no `_test.go` hits for `roleFromFlag`, `agentCmd`, `no valid agent name` or `RoleRule`. The seam exists: open rows in `TestCommand_Run_FlagValidation` at `internal/shuttlecli/cli_test.go:63-87`, and `openRecorderEngine` at `cli_test.go:936-979`. Rows such as `--role "My Agent"` → `OpenSpec.Role == "my-agent"`, and `--role my_agent` → the exact refusal text, would pin it.

2. **MEDIUM, design.** `internal/shuttlecli/agent.go:47-105` (the `agentCmd` RunE). The new verb has no test. Nothing fails if:
   - it maps the wrong geometry field into Inputs (for example NameSlug or ParentName);
   - it drops the language skills;
   - it calls `Open` in place of `StartSession`;
   - it mis-wires its `--name` or `--model` refusal.

   Evidence: there is no test reference to `agentCmd` under `internal/shuttlecli`. The cli_test fake-runner seam at `cli_test.go:936-979` is available.

3. **LOW, design.** `internal/operatoragent/spec.go:303` and `:311`. `TestSessionSpec` asserts neither directive. It never checks that the task row renders the parent variant naming `ly:orch`, or that the prime row renders the no-parent variant. No row gives a PATTERN overview, so a dropped or swapped ParentName, or a pattern directive that never fills, passes. Evidence: in `internal/operatoragent/spec_test.go:411-416` the prompt checks are only hub, path, worktree, place and the edit directive.

Tool calls:
1. Grepped the agentconfig, shuttlecli and contracts/stencils tests for `roles`, `agentCmd`, `roleFromFlag` and `roleOpeningStencils`, to learn which existing tests reach the new role, verb and stencil. My context held only the cli_test.go line hits.
