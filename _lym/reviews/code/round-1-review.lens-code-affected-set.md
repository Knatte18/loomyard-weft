```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, these are the callers, readers, docs, stencil headers and rule documents I found, and whether the diff leaves each correct.

**Per change: what it affects, and whether that is covered**

- **`stencilstore.ReadForEngine`, and `ReadSet`/`readInclude` resolving engine blocks inside plain includes and trimming the final newline:**
  - The producers that switch to it all pass an engine: darn, gatefixer, weaver batch, every webster template, twilldirective. Covered.
  - The seat path (`seatengine/prompt.go:146,202`, `table.go:293`) passes `seat.stencilEngine(defaultEngine)`. Covered.
  - `stencilstore/doc.go` is updated, and so is the seat test whose error wording moved (`table_test.go`). Covered.
  - The callers that still use `stencilstore.Read` read no stencil with an engine include. Covered.
  - `merriambase.go` deliberately keeps a raw `Read`. Covered.
- **`MasterTemplate`, `composeForkTemplate`, `composeRecoveryTemplate`, `composeVerifyFixTemplate`, `RenderForkPrompt`, `RenderVerifyFixPrompt` and `cardGateFixerParagraph` gaining `engine`:**
  - All production callers (`beginbatch.go:359`, `runlevel.go:729`, `render.go`) and the test helpers (`template_test.go`, `export_test.go`) are updated. Covered.
- **`BeginDeps.Engine`:** `webstercli/beginbatch.go` fills it from `c.roles`/`c.engines`, which `wiring.go:207,343` sets. The test fixture is updated. Covered.
- **`twilldirective.Values.Engine`:** both callers (`loomengine/twill.go:88,133`) set it. The package doc and test are updated. Covered.
- **`makeChangeList` taking `engine`:** its one caller, `DarnTable`, is updated.
  - The darn fragment headers are updated. Covered.
  - The `markers_test.go` darn rows are updated. Covered.
  - The `DarnInputs.SubagentModel` doc is not updated (F3).
- **`subagent_model` moving from a top-level marker into the Claude engine blocks (loom-template-discussion, loom-template-plan, raddle-template-agent):**
  - The headers of those three stencils are not updated (F1).
  - The `loomengine/template.yaml` key description is not updated (F2).
- **`SeedEditDirectiveCopies` renamed to `SeedEngineBlockCopies`:** every caller is updated, and no stale reference remains in Go or md outside the research and `_lym` folders. Covered.
- **Fork audit message ("Agent tool call" becomes "nested spawn call") in burlerengine and websterengine:**
  - `cluster_test.go`, `smoke_cluster_test.go` and `fix_test.go` are updated. Covered.
  - The `websterengine/audit.go:44` comment on `ClassNestedAgent` still says "Agent tool call". It is a comment, not a literal, and it is accurate for the Claude transcript field. Not a defect.
- **Twill relaunch wording ("died before its lens spawn call"):** `plan_test.go` and `twilldirective_test.go` are updated. Covered.
- **`verify-module-wide` finding text:** `validate_test.go` is updated. Covered.
- **`modeRules` and the drift judge mode text:** no test asserted the old phrase, so nothing needs updating. Covered.
- **Await help texts (`shuttlecli/await.go`, `awaitforks.go`, `loomcli/parentreview.go`, `renumber.go`):** no reader needs updating. Covered.
- **Declared PATTERN change:** `PATTERN.md` carries the declared `PATTERN-stencil-ownership` text verbatim. The background's Engine blocks section states the rule, the Codex vocabulary and the scan, and the Edit directive section still says the engine copies are the directive's only statement (D3). Covered.
- **Stencil wording, checking whether any rule was dropped:**
  - The Claude blocks keep the "never pass a `name`" reason and the whole await-forks ladder verbatim.
  - The darn/loom/raddle subagent passages keep the effort, skills and one-message launch.
  - "spawn no fork or subagent" replaces "never call the Agent tool" with the same meaning.
  - "with the Read tool" and "with Write" were dropped, and the edit directive already carries those.
  - No rule is weakened.
- **Commit messages against the diff and summary:** the claims match the diff. No `Pattern-Departure:` trailers are present.

**Findings**

- **F1 — LOW — consistency — `contracts/stencils/loom/loom-template-discussion.md:23`, `contracts/stencils/loom/loom-template-plan.md:21`, `contracts/stencils/raddle/raddle-template-agent.md:5-6`.**
  - The stencil headers still declare `subagent_model` as a required top-level marker of the stencil, and raddle lists it under "Every marker below is a top-level {{.X}} substitution".
  - The diff moved that marker into the Claude `subagent-lookups` engine block, so the stencil no longer carries it, and a Codex render has no such marker.
  - The darn fragment headers were updated the same way in this diff ("the {{.subagent_model}} marker its Claude engine block declares"); these three were not.
  - Evidence: the header lines read above, against the diff's removal of `model: "{{.subagent_model}}"` from these stencils (diff2.txt lines 228, 293, 318).
- **F2 — LOW — consistency — `internal/loomengine/template.yaml:9`.**
  - The `subagent_model` key description still says the writers "pass [it] as the Agent tool's model when they hand lookups to fresh subagents", and that the three stencils "read it".
  - Since this diff, only the Claude engine copies read it. A writer on Codex ignores it and spawns on the session's model, as the Codex blocks' headers state ("each fresh subagent runs on the session's own model").
  - The operator-facing config description now misstates what the key does on a Codex launch.
  - Evidence: `template.yaml:9`; `engine/codex/subagent-lookups.md:5` and `engine/codex/darn-subagents.md:5`.
- **F3 — NIT — consistency — `internal/darnengine/spec.go:65`.**
  - The `DarnInputs.SubagentModel` doc still says "the model name the writer passes to the Agent tool", while `makeChangeList`'s updated doc in the same file says "only the Claude wording names that model".
  - Evidence: `spec.go:65` against `spec.go:152-154`.

Outside this lens, for the handler: `webster-template-master.md:92-97,147,188` still tells Master about background completion notifications, a backgrounded Bash command and `SendMessage`. Those are Claude Code concepts that are not on the closed list, and a Codex Master would get them unchanged.

Tool calls:
1. Grepped the stencil headers for `subagent_model`, the doc comments naming `stencilstore.Read` or Claude terms, and stale renamed names. My context lacked the stencil headers and package docs outside the diff.
2. Read the loom and raddle header spans and the package docs on stencil reads, to confirm what those headers and docs state.
3. Grepped the seatengine engine plumbing at render, the `subagent_model` config docs and the `DarnInputs` field doc. My context lacked the render-path callers and `template.yaml`.
