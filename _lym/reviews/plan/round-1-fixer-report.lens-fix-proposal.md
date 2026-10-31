```yaml
lens: fix-proposal
outcome: reported
```

Task: test the review proposals the fixer applied as written, looking for cases where a proposal is wrong or breaks something elsewhere in the plan.

**1. MEDIUM — card 6, F4: the new `TestPerchHardCap` row duplicates an existing row and leaves that row stale.**
- The changed text is: "`TestPerchHardCap` gains a row in which a `tapestry.yaml` `plan_hard_max_bounces` below loom's default is the cap `perchHardCap` returns for Plan-Bouncer."
- The test already has a Plan-Bouncer row, "a worktree plan cap" (`internal/loomcli/circling_test.go:199-206`). It seeds `plan_hard_max_bounces: 6` into `loom.yaml` with `seedLoomConfigWithKeys` and expects `wantRaise` to be `lym config loom --set plan_hard_max_bounces=&lt;n&gt;`.
- After card 6 that key no longer lives in `loom.yaml`, and the way forward goes through `TapestryConfigSpec()`. The existing row's seed and expected raise text are both wrong, and card 6 never says to rewrite them.
- Once that row is rewritten onto `tapestry.yaml`, it is the same case as the new row, so the new row is redundant.
- What should change: rewrite the existing row, seeding `tapestry.yaml` and expecting `lym config tapestry …`, instead of adding a new row.
- The wording "below loom's default" is also unclear. `LoadTapestryConfig` refuses a `plan_hard_max_bounces` below the budget it resolves, so a value chosen below that budget makes the row a refusal, not a cap.

**2. LOW — card 6, F5: the `TestDispatchLayers` row still seeds the board with loom's template.**
- The changed text is: "`TestDispatchLayers`' local-layer row sets `plan_reviewer_model` on module `tapestry` and reads the local `tapestry.yaml`."
- The row also sets `seedBoard: loomTemplate()`, and the harness calls `seedModuleConfig(t, fx.board, row.module, row.seedBoard)` (`internal/configcli/configcli_test.go:1678-1679`).
- With the module switched to `tapestry`, loom's whole template gets written as the `tapestry` module's base. The strict tapestry load would then hit loom's keys.
- The card should also say the row's `seedBoard` becomes the tapestry template.

**3. LOW — card 9, F6: the cwd-resolution background file keeps a stale sibling line.**
- The changed text is: "`pattern/PATTERN-cwd-resolution.md`'s tiers end at `preflight.Check`."
- The same Tiers block says "An orchestrator needs tier 3" (`pattern/PATTERN-cwd-resolution.md`, the line after "Three tiers…"). With `CheckSeed` gone there is no tier 3, and the proposal does not cover that line.

Tool calls:
- Read `circling_test.go:196-232`: I needed `TestPerchHardCap`'s existing rows, and my context held only the grep hits.
- Grepped `envkit.go` for `TemplateSetting`: to check whether `engineMinimalConfig`'s `tapestry.implementer` resolves once `tapestry` is registered. It does, through `configreg.Lookup`, so no finding.
- Grepped for callers of `envkit.NilSeams` and for the cwd-resolution text in `PATTERN.md` and its background file: to check the F3 and F6 sites.
- Grepped `configcli_test.go` for `seedBoard`: my context had only the row's lines, not the harness's seeding step.
