```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked each decision, criterion, constraint and helper branch for the card and the planned row that would fail if it broke.

**Findings**

1. **MEDIUM, D5 criterion "a task's `run_settings` holding `code_reviewer_fan` … is admitted".**
   No planned `TestPrepareRunSettings` row sets `loom.code_reviewer_fan` directly.
   Card 1 admits only the moved darn keys and the renamed `webster_fan` and `webster_reviewer_fan`.
   The `TestModules_LocalLayerRefusals` row covers the local layer, not `run_settings`.
   Evidence: `internal/configreg#TestPrepareRunSettings` as Card 1 plans it; `internal/configreg/runsettings_test.go:23-298` has no such row.
   Fix: add the row `{"loom": {"code_reviewer_fan": ""}}`, admitted, with `wantDocs` `code_reviewer_fan: ""`.
2. **MEDIUM, D6 criterion "a pair seeded from a `light` profile loads `code_reviewer_fan` as one empty entry".**
   Card 2 argues this from three separate tests, and none loads a pair-local `code_reviewer_fan: ""` over a hub or template list value.
   The real stack is the hub's `["review-code", ""]` with a local `""` on top.
   `TestLoadConfig_Loads`' row "empty fan keys never read a malformed lenses.yaml" sets the key in the base file, not the local layer.
   So "the local scalar replaces the hub list whole" is never pinned for this key.
   Evidence: `internal/loomengine/config_test.go:168-176`; `writeLocalLoomConfig` at `internal/loomengine/config_test.go:516` is used only in refusal rows.
   Fix: add a `TestLoadConfig_Loads` or `TestLoadConfig_Refuses`-sibling row that writes `code_reviewer_fan: ""` to the local file over the template board and expects `CodeReviewerFan == lenses.FanList{""}`.
   This also replaces the admitted-load half that Card 1 drops from the "a local fan key is refused" loop.
3. **MEDIUM, Constraint "D6's value survives reconcile".**
   No card carries or tests the claim that configsync's former-value strip, which lists `code_reviewer_fan: ""`, never removes the key from a pair's local file.
   Fidelity to the answer key requires every Constraint to be carried by a card.
   Evidence: `internal/configsync/configsync.go:69-72` (`formerLoomValues` strips `code_reviewer_fan: ""`); none of Cards 1–5 names it.
   Fix: Card 2 gains a row in configsync's reconcile tests where a pair-local loom file holding `code_reviewer_fan: ""` keeps it after a reconcile.
   Failing that, Card 2 must at least cite the existing test that pins "no strip on a local file".
4. **MEDIUM, D1 criterion "`board-scope`'s prose … reports only contradictions with the board … and ends in its list".**
   Card 3's `wantSentences` pins "that is at most six tool calls", "class `decision`" and "Your list:".
   No sentence pins the report-only-contradictions rule, and "contains 'Your list:'" passes even if text follows the list, such as an emphasis line.
   Evidence: Card 3, Tests bullet on `wantSentences`; `internal/lenses/config_test.go:321-335` is a contains check only.
   Fix: add a sentence such as "reports only a contradiction with the board", and assert the normalised text's last sentence opens with "Your list:", or assert that it does not end with the emphasis line.
5. **LOW, D1 and D3 criteria "the template defines neither `goal-scope` nor `board-fit`" and "no `code-pattern-fit`".**
   Only the lens count (37 or 38) pins the deletions, so a kept old lens offset by a missing new one would pass.
   Evidence: Card 3 and Card 5 `wantLenses`; `internal/lenses/config_test.go:253-256`.
   Fix: assert `cfg.Lenses` lacks `goal-scope`, `board-fit` and `code-pattern-fit`.
6. **LOW, D5 criterion "one holding any of the other five fan keys is still refused" through `run_settings`.**
   The planned run-settings rows refuse only `discussion_reviewer_fan` (via `discussion_fan`) and `code_fixer_fan` (via `review_fix_fan`).
   `plan_reviewer_fan`, `discussion_fixer_fan` and `plan_fixer_fan` are pinned only at the local layer.
   Both routes share `ValidateLocal`, so this is low risk.
   Evidence: `internal/configreg/runsettings_test.go` rows "a retired loom fan key…" and "a retired darn fan key moves to loom's code_fixer_fan…".
7. **LOW, helper branch "`emphasisLineGaps` returns names sorted".**
   No planned `TestTemplateChecks_FlagBrokenTemplates` row has two failing names, so the sort is unreached.
   Evidence: Card 3, `TestTemplateChecks_FlagBrokenTemplates` rows.
   Fix: drop "sorted" from the Intent, or add a two-gap row.

**Per decision, criterion and branch**
- D1 fans: Card 3, `wantOrders`. Prose: Card 3, `wantSentences` (partial, findings 4 and 5). Exemption: Card 3, `TestTemplateChecks_FlagBrokenTemplates` rows. Header comment: Card 3, prose only.
- D2 fans: Card 4, `wantOrders`. `way-forward` emphasis line: Card 4, via `emphasisLineGaps`. Writer check: Card 4, `writerFanGaps` rows.
- D3: Card 5, `wantOrders`, the csharp exact-order assertion and `wantSentences`.
- D4: Card 5, the "Test economy" absence check and `wantSentences`.
- D5 local layer: Card 1, `TestModules_LocalLayerRefusals` and `TestLoadConfig_Refuses`. `run_settings` direct: none (finding 1). Moved keys: Card 1, `TestPrepareRunSettings` rows.
- D6 blocks: Card 2, `TestTemplateLadder` and `TestTemplateDocsProfile`. Configreg check: the existing `TestProfilesCheckLoad`. Seed and load: partial (finding 2). Solo round: existing `EmptyStringSettingRecordsASoloRound`.
- D7: Cards 1, 2 and 5, prose only.
- Carry-over `d2-way-forward-loses-emphasis-exemption`: Card 4.
- Reconcile constraint: none (finding 3).
- `emphasisLineGaps` branches: exempt, generic, passing and failing each have a row; the sort has none (finding 7). `writerFanGaps` branches: direct, absorbed, reviewer-only and missing each have a row.

**Tool calls**
- One grep of `internal/shedrecipe` tests and `internal/loomengine/config_test.go`: I confirmed the `EmptyStringSettingRecordsASoloRound` row exists and that no loom test loads a local fan key as admitted. My context held only the glyph, not the row or the local-layer load tests.
