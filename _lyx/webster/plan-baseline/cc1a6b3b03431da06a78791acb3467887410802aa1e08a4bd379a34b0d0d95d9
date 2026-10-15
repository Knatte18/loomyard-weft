# Card 4 — Pin treadle's templates to the values Go parses

**Edit:**
- `internal/treadleengine#TestShippedTemplates`
- `internal/treadleengine#requireContains`

**Delete:**
- `internal/treadleengine#requireHandoffMaintenanceRules`
- `internal/treadleengine#requireQuotedRationaleRule`

**Uses:**
- `internal/treadleengine#JudgeProgressing`
- `internal/treadleengine#JudgeCircling`
- `internal/treadleengine#JudgeContinue`
- `internal/treadleengine#JudgeStop`
- `internal/treadleengine#JudgeUncertain`
- `internal/treadleengine#judgeHeader`
- `internal/treadleengine#handoffHeader`

**Intent:**
`TestShippedTemplates`' "states load-bearing rules" subtest pins static prose of one stencil each, which D1 drops; what stays is the stencil naming Go-declared values, read from the Go definitions in this package.
Replace the subtest with "names the values Go parses": each judge row lists the values its stencil must name, built from Go, and the subtest fails for each one the stencil text lacks:
the circling judge names `string(JudgeProgressing)`, `string(JudgeCircling)` and `string(JudgeUncertain)`; the milestone judge names `string(JudgeContinue)`, `string(JudgeStop)` and `string(JudgeUncertain)`; both judges name the `yaml` tag of `judgeHeader`'s `Rationale` field and of `handoffHeader`'s `CoversRounds` and `Ledger` fields, each read through `reflect` (`rationale`, `covers_rounds`, `ledger`).
The targeting row names no Go-parsed value, so it carries no list, and the subtest runs only for a row with a non-empty list, so the targeting row reports no vacuous pass.
The `phrases`, `quotedRationale` and `handoffRules` fields go, and with them every prose phrase ("clear, citable evidence", "when in doubt", "## Themes", "EXACTLY TWO", "EXACTLY ONE", "free-form prose" and the others), and the two helpers that held the handoff-maintenance and quoted-rationale prose are deleted.
The "fills with all markers" subtest stays as it is.

The test's doc comment and the file comment heading `template_test.go`, above `requireContains`, stop describing load-bearing statements as substring assertions: the file pins that each judge template names the verdict words and frontmatter keys this package parses, and that each template fills with its required markers.

**ImpactSummary:** test-only, confined to `internal/treadleengine/template_test.go`; the two deleted helpers are called only by `TestShippedTemplates`.
