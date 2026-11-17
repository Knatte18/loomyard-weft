# Fixer report — code review round 1

All seven findings are fixed as proposed, each in its own commit.
The gate (`lym gate test` over `contracts/stencils`, `internal/darnengine`, `internal/twilldirective`, `internal/websterengine` and `internal/loomengine`) passed after the fixes, and again over `contracts/stencils` and `internal/loomengine` after the self-check fixes.

## Changes

- **F1** — added one row per producer that resolves the launch to Codex and asserts the Codex wording and the absence of the Claude copy:
  - `internal/darnengine/spec_test.go`: `TestDarnSpec_PromptFollowsWriterEngine`, a writer on the `sol` alias.
  - `internal/twilldirective/twilldirective_test.go`: a row with `Values.Engine` set to `codex`.
  - `internal/websterengine/beginbatch_test.go`: a row with `BeginDeps.Engine` set to `codex`.
  - `internal/websterengine/runlevel_test.go`: a `TestRun_MasterSpawn` row with Master on Codex and Recovery on the fake engine, asserting the Master prompt and `verify-fix.md`.
  - The loom Discussion and Plan producers (`discussiontable.go`, `plan.go`) have no Codex row: the review's fix names the darn, twill and webster test files only.
- **F2** — `contracts/stencils/engine/codex/fork-wait.md` now uses the review's wording: `wait_agent` with a `timeout_ms`, counting one final answer per fork.
- **F3** — new `engine-fork-message` block with a Claude copy (`SendMessage` by agent id) and a Codex copy (`send_message` to the task name `spawn_agent` returned), both registered in `contracts/stencils/stencils.go`; the Master stencil includes it at the `later_target_removed` rung.
- **F4** — the headers of `loom-template-discussion`, `loom-template-plan` and `raddle-template-agent` say the Claude engine block declares `subagent_model`; raddle's top-level marker list drops it.
- **F5** — `internal/loomengine/template.yaml` line 9 says the writers on Claude pass the model and a writer on Codex ignores the key.
- **F6** — `missingEngineBlockCopies` reports a stencil that fails to parse, instead of skipping it.
- **F7** — `DarnInputs.SubagentModel` doc reworded as the review proposes.

## Self-check

- fix-code-correctness, LOW: the doc of `missingEngineBlockCopies` omitted the parse-error line. Fixed as proposed.
- fix-affected-set, LOW: `internal/loomengine/config.go` `SubagentModel` doc still named the Agent tool, the twin of F5 and F7. Fixed as proposed.
- fix-consistency, LOW: the Master stencil's "completion notification" wake wording is Claude Code's, outside any engine block. Deferred, see below.
- fix-consistency, LOW: no list of engine blocks outside `stencils.go`. Non-finding, no change.
- fix-affected-set, unchecked note: `burler-cluster-fan` and `burler-fix-self-check` have no Codex row. Not a finding, since F1 named other producers.
- fix-proposal: no findings.

All four forks returned a report. Their files sit beside this one as `round-1-fixer-report.lens-<lens>.md`.

## Gate repair

The gate refused two comment lines of 120 characters or more, the `SubagentModel` docs in `internal/darnengine/spec.go` and `internal/loomengine/config.go`.
Each is now two lines, one sentence per line.

The verify run then failed `TestAgentMessaging_NoEngineTool` in `cmd/lym`:
F3's Claude `fork-message.md` block names `SendMessage`, which the test's allowlist admitted only in the Master stencil.
The allowlist row in `cmd/lym/agentmessaging_test.go` now admits that line in `contracts/stencils/engine/claude/fork-message.md`, since the Master stencil no longer carries it.

## Disputed

None.

## Deferred

- fix-consistency's wake-wording note: how a Codex Master is woken after a fork finishes is a wording decision on the Master stencil's loop beyond F3, which only named `SendMessage`. It needs the operator's call on the Codex wake model, and the closed term scan does not cover it.

## Rulings

None.
