```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering everything outside the touched lines that the diff's changes force.

**Per change: the affected set and whether it is covered**

- **`editdirective.Directive(stencilsDir, engine)` replacing `Directive(stencilsDir)` and `EngineDirective`:** covered.
  - Every production caller passes an engine: tackler, treadle judge and targeting, sampler, webster (×3), landingshed, seatengine, gatefixer, loomcli driver, dobby, arbiter, friction, mergeresolve, weaver and bouncer (×2).
  - No Go comment or doc keeps the old signature or names `EngineDirective`.
  - The test fixtures were moved to the engine block name.
- **Removing the `edit/edit-directive.md` stencil, its `EditDirective` embed and its registry row:** mostly covered.
  - `stencils.go` drops the embed and the row, and the guard in `parentdirective_test.go` now exempts the engine copies.
  - `pattern/PATTERN-stencil-ownership.md` § Edit directive is updated.
  - `docs/research/**` still names the old path, but those are point-in-time reports and are exempt.
  - Not covered: four opening-stencil banners still call the marker "the shared edit directive" (F1).
- **The engine copies' banners and the `stencils.go` doc comments for both engine copies:** covered.
  - The burler assist templates' banners ("in the assistant engine's own wording") are still true.
- **`Spec.PromptEngine` and `Spec.HandWrittenPrompt`:** covered.
  - Every production `shuttleengine.Spec` literal that sets `Prompt` also sets `PromptEngine`: bouncer, treadle, sampler, friction, mergeresolve, landingshed, dobby, loomcli driver, arbiter, tackler, weaver, webster (×3), gatefixer, seatengine and orch.
  - Only `shuttlecli/run.go` sets `HandWrittenPrompt`.
  - `gatefixer.Spec` gained `PromptEngine`, and all three of its producers fill it: gatecli, loomcli and weaver/webster.
- **The `EngineNamer` seam added to each producer's shuttle interface:** covered.
  - Production implementations: `Runner`, `EngineSet`, `RunnerShuttle`, `runnerDriverStarter` and `runnerTidyShuttle`.
  - Test kits: shedfake, envkit and the per-package fakes.
  - Wiring: the loomcli sampler gets `Engines: engines`, and landing `Sessions` is the runner.
- **`CheckPromptEngine` in both `Prepare` methods:** covered. The engine name constants are shared, with claudeengine folding `turnHookEngineName` into `engineName`.
- **Declared PATTERN changes:** covered.
  - The only declared change is the stencil-ownership background's edit-directive section (D3), and it landed in `pattern/PATTERN-stencil-ownership.md:59-67`.
  - The `PATTERN.md` provider-seam line and the new "Prompt engine" background section are additions that state D4. No declared entry is missing.
- **Commit messages and departures:** covered. The four range commits carry no `Pattern-Departure:` trailer, so there is nothing to judge.
- **Repository rule `PATTERN-code-conventions`:**
  - "No hard-coded choice": the new `orchEngine = "claude"` literal is not covered (F2).
  - "Named constants": the duplicated `"edit-directive"` literal in stencilkit is not covered (F3).

**Findings**

- **F1 — LOW, consistency.** Four stencil banners still describe `{{.edit_directive}}` as "the shared edit directive rendered by internal/editdirective", and no shared directive exists any more.
  - The stale lines are `contracts/stencils/loom/loom-template-arbiter.md:17`, `contracts/stencils/shed/shed-template-tackler.md:16`, `contracts/stencils/beam/beam-template-board-tidy.md:6` and `contracts/stencils/shed/shed-template-driver.md:13`.
  - The diff left them unchanged, while `pattern/PATTERN-stencil-ownership.md:64` and `internal/editdirective/doc.go` now say "No shared copy exists".
- **F2 — BLOCKING, decision.** `internal/orchcli/remotecontrol.go:42` adds `const orchEngine = "claude"`, a caller naming an engine.
  - `pattern/PATTERN-code-conventions.md` § No hard-coded choice says "no caller names an engine" and "An existing literal … is debt: a change that touches it … never adds another".
  - The literal carries out D4's "The orch spec names the Claude engine … explicitly", so the constraint conflicts with the answer key. Under the prompt's rule the constraint wins, and I am reporting the conflict rather than picking a side silently.
  - A fix that satisfies both would read the orch engine from orch config, with Claude as the shipped default. Otherwise the parent records the exception with `lym loom decision add`.
- **F3 — LOW, consistency.** `internal/testkit/stencilkit/stencilkit.go:40` declares a second `editDirectiveBlock = "edit-directive"`, repeating the name that `internal/editdirective/editdirective.go:18` (`EngineBlock`) owns. Its comment justifies the copy by an import cycle.
  - `PATTERN-code-conventions` § Named constants says "one named constant, declared once and referenced everywhere, never a repeated literal".
  - A rename of `EngineBlock` would leave the test kit seeding a stale name with nothing to catch it.
  - The same file and `internal/testkit/shedfake/shedfake.go:79` also hard-code the default engine `"claude"`.

Tool calls:
1. A repo-wide grep for `edit-directive`, `EngineDirective`, `edit_directive`, "shared edit directive" and the pattern/docs edit-directive wording. The manifest's grep covered only Go and md under some paths and did not catch stencil banner comments.
2. A grep for stale `Directive(stencilsDir)` signatures in comments, for orphan-stencil handling in stencilstore, and for assistant-specific edit-directive wording in Go docs. These were not in the manifest.
