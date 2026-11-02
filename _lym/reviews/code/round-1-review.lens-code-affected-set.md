```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change in 9d996439, I list what reads or asserts it and whether the diff leaves it right.

**Per change**
- **`Outputs.recipeOf` and `Board.RecipeOf` (one resolution point).**
  - Readers of a task's `Recipe` field in non-test code: `battencli/wire.go:526` (`boardRecipe`), `loomcli/peers.go:295` and `:250`, `boardengine/config.go:104`, and the `boardengine` validation, owner-set, replay and run-lock sites.
  - Only `config.go:104` tests an empty `Recipe` and falls back to a default. That is covered.
  - The remaining reads do not resolve the field. They are validation (`task.go:316,325`), a copy or compare (`replay.go:153`, `runlock.go:156,198`), or an owner-set write (`ownerset.go:194`).
  - `peers.go:259` keeps a `shedrun.RecipeTapestry` fallback. It is now reached only for an empty self-run seed recipe, as the change description says. Covered.
- **`recipeCell(t, out Outputs)`.** It is unexported, and both call sites (`render.go:468`, `:490`) pass `b.out`. There are no other callers. Covered by `TestRenderRecipeCellTakesTheDefaultRecipe` and `TestRenderReadmeGolden`.
- **`ReadBoardType` doc (`battenshed/deps.go`).**
  - It now matches the closure at `battencli/wire.go:1304-1316`, which resolves through `boardRecipe` and `RecipeOf`.
  - The closure's own comment at `wire.go:1303` agrees.
  - `seamchild.go:65` is the only consumer. Covered.
- **`loomcli/doc.go` peers paragraph.** It agrees with the board closure at `peers.go:295`, which writes the resolved recipe back into `Task.Recipe`, and with the `peerRecipe` comment at `peers.go:246`. Covered.
- **README default sentence.** `README.md:61` is the only README statement of `default_recipe`. Covered.
- **Other docs and the template.** Four sites outside the diff still name the setting or its values. Findings 1 and 2 cover them.
- **PATTERN entries.**
  - The diff declares no PATTERN change, and I found none required.
  - Config-layers: no new reader of `board.yaml` is added.
  - Test-speed: `t.Parallel` on `TestLoadConfig_DefaultRecipe` matches the rule, since `writeBoardConfig` uses `t.TempDir()` and no process-global state.
- **Commit message.** The commit has no `Pattern-Departure:` trailer and needs none. Its claims match the diff.

**Findings**

1. LOW, consistency. `internal/boardengine/template.yaml:3`.
   - The comment on `default_recipe` still lists "darn, tapestry or sampler".
   - The same commit removed that list from `Config.DefaultRecipe`'s doc (`config.go:32`) because it goes stale apart from the field.
   - The template's comment is the user-facing copy of the same enumeration, and it was left.
   - Evidence: `template.yaml:3` against the `config.go` diff hunk.

2. LOW, consistency. `docs/overview.md:364`, `docs/overview.md:680` and `plugins/ly/skills/board/SKILL.md:52`.
   - The decision record asks for "docs that state the setting once".
   - The diff leaves the `README.md:61` sentence and three more statements of the default and its meaning: `overview.md:364` ("`darn` in the template"), `overview.md:680` ("empty meaning `board.yaml`'s `default_recipe`") and `SKILL.md:52` ("`darn` unless the hub changed it").
   - The README fix removed only the duplicate inside the README.
   - A reasonable reading is that the other docs serve different readers. Weigh this against the record's wording; it is at most LOW.

No MEDIUM or BLOCKING findings. No other symbol, doc or fixture depends on the changed signature or the changed comments.

**Out of range, not a finding:** the `peerRecipe` doc (`peers.go:247`) names `boardRecipe` from another package. The commit under review did not touch that line.

**Tool calls**
1. `grep -rnE '\.Recipe\b' --include=*.go internal cmd`, excluding tests and seed/req fields. My context held only the narrower grep from step 1, which did not list every reader of a task's `Recipe` field.
