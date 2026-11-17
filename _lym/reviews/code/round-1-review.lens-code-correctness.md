```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each changed function and branch with concrete inputs.

**List: function or branch, the case run, verdict**

- **`stencilstore.writeEngineBlocksInPlace`**
  - Body with no leading comment: header is "" and `rest` is the whole text. Correct.
  - Body that opens with whitespace and then a comment: `stripLeadingComment` returns a suffix of the text, so `text[:len-len(rest)]` is the comment. Correct.
  - Header that spells `{{template "engine-absent"}}`: not replaced, not parsed, kept. Correct.
  - Engine "": `readInclude` refuses with the "no engine was named" way forward, and the error is returned. Correct.
  - Engine "gemini": `*MissingEngineBlockError` comes back through `readErr`, and later matches are skipped. Correct.
  - Trimmed action `{{- template "engine-tool"}}`: the regex misses it, `IncludeNames` sees it, and it is refused. Correct.
  - A block whose copy includes another block: the second pass refuses it. Correct.
- **`stencilstore.ReadForEngine`**
  - Unreadable stencil: `Read`'s error is returned. Correct.
- **`stencilstore.readInclude`**
  - Engine block: the copy keeps its header and loses its final "\n". `attachIncludes` strips the header (`stencil.go:113`), and the in-place path strips it too. An inline block in a heading, such as `## Never use {{template "engine-question-tool"}}`, renders "## Never use `AskUserQuestion`". Correct.
  - Plain include, engine "", no engine block: same text as before. Correct.
  - Plain include holding an engine block, engine "": refused, with the plain include's name as the stencil. Acceptable.
  - Plain include that does not parse: it now fails inside stencilstore; the seat test row was updated. Correct.
- **`seatengine.Validate` with `seat-directive-assistant-chair`, now an include carrying an engine block**
  - Codex chair: the block resolves in place, so the nested-include check sees no nested include. Correct.
  - Assistant seat with a missing copy: still skipped. Correct.
- **`darnengine.DarnTable` / `makeChangeList`**
  - Engine comes from `NewModelChoice(resolved).Engine`, which `modelspec` always sets: aliases require an engine and the escape form carries one. Correct.
  - Codex: `Fill` gets an unused `subagent_model` value; `Fill` does not refuse extra values. Correct.
  - Claude: the marker is filled. Correct.
  - The fragment's header, which mentions `{{.subagent_model}}`, is kept by `ReadForEngine` and stripped by `Fill`. Correct.
- **`gatefixer.RenderPrompt`**
  - Frame and body read for `in.Engine`. The frame now includes no block, so an empty engine still reads it.
  - Every caller (`weaverengine/verify.go`, `websterengine/verifyfixer.go`, `loomcli/gatefixer.go`, `gatecli/detachfixer.go`) already passes `Engine` for the edit directive. Correct.
- **`twilldirective.Directive` / `fillPart` / `judgePart`**
  - Empty `Values.Engine`: `ReadForEngine` refuses on `twill-directive`'s fork-spawn include, and the error names the stencil.
  - Both loom callers pass `resolveModelChoice`'s `.Engine`, which is never empty. Correct.
- **`loomengine.planLensDirective` / `discussionLensDirective`**
  - The engine matches the seat's `ModelChoice` engine, so the directive and the seat render for the same engine. Correct.
- **`webstercli.beginDeps`**
  - Both wirings (`wiring.go:207` and `wiring.go:343`) set `c.roles` and `c.engines` before any verb runs.
  - A missing `RoleMaster` is refused before anything is mutated. Correct.
- **`websterengine.Run`**
  - The `RoleMaster` lookup moved ahead of `RenderVerifyFixPrompt`. A missing role now fails earlier, with no side effect before it; `engine` is reused for `RenderMasterPrompt`. Correct.
- **`websterengine.BeginBatch` / `RenderForkPrompt`**
  - `deps.Engine` is Master's engine. The fork inherits Master's session, so that engine is right. Correct.
- **`RenderVerifyFixPrompt` / `composeVerifyFixTemplate`**
  - Master's engine, since the fixer is an in-session fork. Correct.
- **`RenderRecoveryPrompt` / `composeRecoveryTemplate`, `cardGateFixerParagraph`, `RenderPlanFixPrompt`, `RenderMasterPrompt`**
  - Each takes the engine of its own launch. Correct.
- **`websterengine.MerriamBase`**
  - Raw read; the line count is engine-independent, the same as before through `MasterTemplate`. Correct.
- **`weaverengine.RenderBatchPrompt`**
  - Reads with `in.Engine`, which was already set for the edit directive. Correct.
- **`burlerengine.forkViolation` and `twillEngineRefusal`, `websterengine.CheckFork`**
  - Message text only; the tests' substrings were updated to match. Correct.
- **`loomengine.modeRules`, `driftengine` mode constants, `planparser.checkVerifyModuleWide`**
  - String changes only. Correct.
- **`stencilkit.SeedEngineBlockCopies`**
  - Fake engine "b": every registered `engine/claude/*` copy is written as `engine/b/*`. Correct.
- **`engineterms_test` `missingEngineBlockCopies`**
  - A body that fails `IncludeNames` is skipped with `continue`. That swallows the error and hides that body's engine includes from the copy check. See F2.
- **`engineterms_test` `goLiteralTermViolations`**
  - The `internal/orch` prefix skips `orchcli`, `orchengine` and `orchroute`.
  - An `Unquote` failure is fatal. Correct.
- **Claude copy of `fork-wait` placed inside numbered steps** (`twill-directive.md` step 3, `burler-fix-self-check.md` step 3)
  - Its second and third lines now render at column 0 inside the list item; the old text indented them three spaces. See F1.

**Findings**

1. **F1 — LOW, consistency.** `contracts/stencils/engine/claude/fork-wait.md:7-8`, as rendered in `contracts/stencils/twill/twill-directive.md:20` and `contracts/stencils/burler/burler-fix-self-check.md:20`.
   - Summary: the Claude `fork-wait` copy's lines "While it answers `ready: false`…" and "When it fails…" carry no indent. In a numbered step they leave the list item's three-space continuation, which the replaced text had (old `twill-directive.md`, diff lines 362-363).
   - Effect: in raw text, which is what the agent reads, they now sit at column 0 between step 3's first line and its indented "Edit nothing…" line. In markdown they are lazy continuation lines. The meaning is unchanged.
2. **F2 — LOW, design.** `contracts/stencils/engineterms_test.go:389-392` (`missingEngineBlockCopies`).
   - Summary: when `stencil.IncludeNames` errors on a body, the error is dropped with `continue`, so that stencil's engine includes are never checked for a copy per engine.
   - Effect: a parse error should fail loudly, for example with `t.Errorf` or a returned violation line. Today the check is silently skipped.

Outside my lens, for the handler: the Codex `fork-wait` copy (`engine/codex/fork-wait.md:6`) says "call `wait_agent` on every fork you spawned". The spike report (`codex-twill-spike-2026-10-24/report.md:97,211`) says `wait_agent` names no agent, so the parent has to count one `FINAL_ANSWER` per child.

**Tool calls**
- Read `internal/stencil/stencil.go`'s `attachIncludes` and `stripLeadingComment`: my context did not show whether an include's leading comment is stripped, which decides whether inline engine blocks render cleanly.
- Grepped the `gatefixer.RenderPrompt` callers and webster's CLI wiring: my context did not show whether every fixer caller sets `Engine`, or whether `begin-batch`'s wiring sets `c.roles` and `c.engines`.
