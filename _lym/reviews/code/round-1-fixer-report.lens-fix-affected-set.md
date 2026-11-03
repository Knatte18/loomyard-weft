```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran the search behind each finding and checked whether the new tests cover every member.

No findings.

- **F1 (seatengine `DefaultEngine` sites):** the search found `engine.go:230` (Run), `resume.go:30` (Probe), `resume.go:94` (Resume), `prompt.go:97` (Prompts) and `prompt.go:129` (CheckMarkers). It also found `prompt.go:177` (composeSeatPrompt, the path `startSeat` takes). The new test covers Run, Probe and Resume, and Run's prompt check covers `startSeat`. Prompts and CheckMarkers were already pinned by existing rows, so every member is covered.
- **F2 (burlerengine engine choice):** `seatEngine` has exactly two callers, `review.go:99` (`opts.Review`) and `fix.go:157` (`opts.Fix`). The new test pins both pairings and the fallback: Review is `claude`, Fix is the default `codex`. The `New` copy of `DefaultEngine` is pinned because the fixer seat stencil includes the engine block. Covered.
- **F3 (explore `ReadSet`):** the explore `ReadSet` at `prompt.go:116` is now covered by the added "explore step" row. The other `ReadSet` sites, `prompt.go:91` (`fillInstruction`) and `prompt.go:175` (fix step), were already covered through the review-step and fix-step rows.

Left unchecked, since the review did not name them and they are not part of any finding's set: the engine arguments of the helper blocks that go through `fillInstruction`. These are `focusDirectiveBlock`, `forkReadingBlock`, `clusterRulesBlock`, `selfCheckBlock` and `renderReviewFormat`. No test puts an engine block inside those stencils. They share the single `fillInstruction` argument path that the review step already exercises.

Tool calls:
1. grep for `DefaultEngine` and `stencilEngine` in seatengine and burlerengine non-test files, to reconfirm the F1 set.
2. grep for `composeReviewPrompt`, `composeFixPrompt`, `seatEngine(` and `ReadSet(` in burlerengine non-test files, to confirm the F2 and F3 sets.
