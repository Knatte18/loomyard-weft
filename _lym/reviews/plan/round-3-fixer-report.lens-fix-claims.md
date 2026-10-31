```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**MEDIUM: card 02 Intent says every unknown-recipe row counts one way-forward clause, but the status row never reaches that count.**
- **Changed text:** `_lym/plan/02-drop-beam-alias.md`, Intent: "Every row of `TestArmFromSeed_RefusalEnvelopeKinds` that seeds an unknown recipe, … asserts that text as its suffix and counts one way-forward clause. The kind-less branch …, the `wantKind == ""` early return, also asserts `wantSuffix` on the envelope's error text when the row sets one".
- **Problem:** The rewrite adds only a suffix check to the kind-less branch. The way-forward count still sits after the early return, so the "unknown recipe on status" row never counts clauses. The sentence still says every such row does.
- **Evidence:** `internal/shedcli/cli_test.go:274-279` returns inside `if tt.wantKind == ""`. The count check at `:293` (`strings.Count(msg, "way forward:") != tt.wantWayForwardCount`) comes after that return, and the status row at `:200-203` sets no `wantWayForwardCount`.
- **Fix:** Either the kind-less branch also asserts `wantWayForwardCount` when the row sets it, and the status row sets it to 1, or the sentence limits the count to the step and loop rows.

All other added claims hold:
- **Card 06, TestMigrateConfig:** an unknown key in a loom layer is only logged and the load carries on (`configengine/config.go:225`, `warnUnknownKeys` returns nil after `logger.Warn`). The `plan_producer`→`plan_lenses` rewrite stays pinned by the row at `config_test.go:923`.
- **Card 06, second TestCheckResolved row:** the hard-below-soft refusal's prefix names `caps.HardKey` (`reviewbudget.go:150`). With `plan_hard_max_bounces: 0` that key is `review_hard_max_bounces`, which loom owns under card 6's routing, so the prefix reads `loom config key "review_hard_max_bounces"`.
- **Card 09:** tapestry numbers drop by one after Preflight, so Friction-Reflect goes from 16 to 15. Plan-Write goes from 6 to 5. The Plan-Review segment shares one number (`names.go:115-117`), so Plan-Burler-Review goes from 7 to 6, which gives `6.6`.
- **Card 10:** Drift-Verify is a single row, so after the move it takes Code-Bouncer's old number and the Code segment takes the next one.
- **Card 11:** darn numbers rise by one: Drift-Baseline 2→3, Code segment 6→7, `[ ] 10. Friction-Reflect`→11. In the fixture (`checklist_test.go:485-495`) Record-Write is not in the history, so `[ ] 2. Record-Write` is right. Wording nit: the "`[ ] 3. Drift-Baseline` through `[ ] 11. …`" range also covers the `[x] 5. Darn-Write` and `[&gt;] 7. Code-Review` lines, which are not `[ ]` rows. Not raised.

Tool calls:
1. Read `checklist_test.go:480-500`, `loomrecipe/names.go` `EntryNumbers` and the tail of `warnUnknownKeys`: the darn checklist fixture, the segment-numbering rule and the warn-or-refuse outcome were not in context.
2. Grepped `CheckResolved`'s refusal format strings in `reviewbudget.go`: the hard-below-soft prefix key was not in context.
3. Read `configengine/config.go:205-250`: to confirm the load carries on after the unknown-key warning.
