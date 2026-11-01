```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: per behaviour, the test that pins it**

1. **`orch-prompt` reaches orch sessions only, including numbered siblings (`orch-2`), and excludes lookalikes (`orchard`):** pinned by `TestRender/OrchPromptReachesOnlyOrchSessions` (`internal/heddleengine/render_test.go:148-179`). Its rows are ClaudeOrch, ClaudeNumberedOrch, CodexOrch, ClaudeWriter, ClaudeOrchLookalike and CodexWriter.
2. **An entry aimed at both `claude-prompt` and `orch-prompt` is delivered once:** pinned by the same test. The `both` entry appears once per row.
3. **Engine-only targets keep their behaviour with an empty role:** pinned by the changed `Passages(f.bolt, "claude", "")` and `("codex", "")` assertions at `render_test.go:378-386`.
4. **`Show("orch-prompt")` prints the orch passages:** pinned at `render_test.go:175-178`.
5. **Claude `Prepare` passes `spec.Role` to the passage reader:** pinned by the `role_reaches_reader` row in `TestPrepare_AppendsStandingPassages` (`prepare_test.go:484-491`). The row asserts the engine and role it passes.
6. **`StandingPassages` and `PassageReader` carry the role argument:** pinned indirectly by the `Prepare` test above.
7. **`applyOrchRules` (`rules_test.go:31-34`):**
   - New entry takes `orchRulesOrder`, and an existing entry keeps its order (`NewEntry`, `ReplaceKeepsOrder`).
   - A blank draft removes the entry (`BlankClears`), and a blank draft with no entry is a no-op (`BlankWithNoEntry`).
   - The unrelated `house` entry is left untouched, as seeded at line 40.
8. **`writeOrchRules` refusals (missing `--from`, unreadable draft):** pinned by `TestWriteOrchRules_RefusesBeforeTheStore` (`rules_test.go:70-82`).
9. **`lym orch rules` end to end (tier1):** pinned by `TestOrchIntegration_RulesStoreInHeddle` (`rules_integration_test.go:20-66`).
   - It covers the envelope fields `slug` and `cleared`.
   - It covers the stored text, order and targets.
   - It covers the removal on an empty draft.
10. **Role stencil markers drop to two:** pinned by `contracts/stencils/markers_test.go` and `TestRenderRoleFile_RefusalsWriteNoFile` (`prompt_test.go:72-98`). The stencil's static prose is exempt.
11. **`RenderRoleFile` loses its rules arguments and the `readRules` placeholder path:** `prompt_test.go:36` and the rewritten `prompt_test.go` cover the new signature.

**Findings**

- **LOW, design.** `heddlecli.ChangeStore` has no test of its own.
  - **Location:** `internal/heddlecli/cli.go:49-58`.
  - **Evidence:** a grep for `ChangeStore` in `*_test.go` found no hit. Only the orchcli tier1 test reaches it, and only on the success path.
  - **Untested:** the error path (a cwd outside a drivable hub; a store change that returns an error saves nothing and starts no sync) and the `warning` return when the sync does not start.
  - **Related gap:** `writeOrchRules` copies `warning` into the envelope (`rules.go`, the `fields["warning"]` branch), and no test sets or asserts it.
- **LOW, design.** The `writeOrchRules` error branch has no test: a `ChangeStore` error returned as `output.Err` (`rules.go`, `if err != nil { return output.Err(out, err.Error()) }`). The refusal test covers only the pre-store refusals (`rules_test.go:70-82`), not a refusal from the store or a non-hub cwd.

I did not check whether the deleted rules-file tests in `rules_test.go`, `prompt_test.go` and `watcher_test.go` left any behaviour uncovered. The diff only shows their removal, and the remaining behaviour is no longer in the code.

**Tool calls**
1. Bash grep of `rules_test.go`, `rules_integration_test.go`, `prompt_test.go` and `markers_test.go` for test names and rows, plus a `ChangeStore` test search: my context held only the diff of the non-test files.
2. Read `rules_integration_test.go`: its assertions were not in the diff I had read.
