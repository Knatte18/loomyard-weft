```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's test changes (F1 to F3) by running concrete inputs through them.

No findings.

- **F1, `TestEngine_EntryPoints_ReadEngineBlocksForTheGeometrysDefaultEngine`:**
  - Run with the default engine `""` is refused at `Validate` (`engine.go:230`), so the NoEngine row sees the refusal.
  - If `startSeat` passed `""`, `Validate` would still pass but `composeSeatPrompt` would fail. The chair would never start, so `awaitStarts(1)` would fail the Codex and Claude rows.
  - If `Probe` or `Resume` passed `""` to `Validate`, the Codex and Claude rows would get a refusal where they expect nil.
  - The Resume subtest releases the chair before the call, so `join` returns at once, and the handle is cleaned up in all rows.
- **F2, `TestEngine_HalvesRenderEngineBlocksForTheirOwnEngine`:**
  - The Review row uses a choice engine of `claude` against a default of `codex`, so the review step and the reviewer prompt must hold the Claude text.
  - The Fix row has no choice engine, so its fixer prompt must hold the Codex text.
  - If `Review` and `Fix` passed each other's opts, both rows would fail.
  - If `seatEngine` dropped its fallback, the Fix row would fail on an empty engine.
  - If `New` dropped `DefaultEngine`, the fixer seat's stencil would be refused (it includes an engine block), so `Fix` would return an error.
  - The Review glob matches exactly one directory because each row has its own root.
- **F3, the explore step added to `TestComposePrompt_RendersEngineBlocksForTheHalfsEngine`:**
  - An empty or wrong engine in the explore `ReadSet` call at `prompt.go:116` would be refused or would render the other engine's text, so the new assertion fails.
  - `reviewExploreFile` is index 0, which matches the file order in `composeReviewPrompt`.
  - Removing the `stencilstore` import is safe, since the only other use of it in that file moved into `seedEngineWords`.
- **Callers:** the fix changed no non-test signatures. `seedEngineWords` and `firstGlob` have no name collisions, and the package compiled in the fixer's gate run.

Tool calls: none; the context already held everything this lens needed.
