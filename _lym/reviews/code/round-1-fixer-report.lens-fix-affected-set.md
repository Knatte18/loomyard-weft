No findings.

Set checked: comments and docs elsewhere that describe the gate rule, the raw go row or "each row's steer".

- F1 (cross-row comparison in a comment): no other row comment in `internal/shellguard` explains itself by comparison to another row. The remaining mentions of "the raw go row" (`rows.go:158`, `rows.go:190`, `doc.go:17`, `claudeengine/settings.go:273`, `claudeengine/doc.go:238`) state a fact and compare no rows.
- F2 (which steers carry the rule): `rows.go:158` already lists the raw go and refuse-raw steers as the ones carrying the rule, matching the corrected `doc.go:18`. No other text claims every row's steer carries it.

Tool calls:
1. `grep` over non-test Go files under `internal` for the finding phrases; the context did not hold those files.
