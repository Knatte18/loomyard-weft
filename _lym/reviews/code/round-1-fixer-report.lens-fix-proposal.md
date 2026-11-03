```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1, F2 and F3 proposals as the fixer carried them out.

- **F1** (`baseline_test.go`, new row "an existing baseline.md is kept under a configured scope, with no agent"): no findings. Without the `os.Stat` guard in `settleConfiguredScope`, the generated report would overwrite the seeded "expected drift". The row's `string(data) != "expected drift"` check would then fail, so the row catches the defect. The row also asserts no agent and one capture start.
- **F2** (`configuredBaselineReport` text and the configured-full assertion): one finding.
  - **LOW**: the doc comment above `configuredBaselineReport` still reads "renders the baseline.md of a scope the drift config set". That is the provenance the fix removed from the body, and the scope may be a parent's or an earlier record. Evidence: `baseline.go:168`, against `settleConfiguredScope`, which renders from `ReadScope`'s recorded scope. Rewording to "renders the baseline.md written without a baseline agent" would fix it.
  - The new assertion "no baseline agent ran" matches the new body text. No other test or file greps the old "drift.scope" text. The grep was for the stencil phrase only; the earlier read of `baseline_test.go` showed "drift.scope" only in the configured-full row.
- **F3** (`drift-template-verify.md:23`): one finding.
  - **LOW**: the added clause is conditioned on "when the scope was configured", but the verify agent has no marker for that. `settleConfiguredScope` also skips writing when `baseline.md` already exists. If a parent or an earlier agent wrote the file, the scope still counts as configured but the file is an agent's report with per-test expectations, so the clause is false for it. Keying the clause on the file's content would be accurate, for example "when the report says no baseline agent ran, it names only the scope and its skipped tests". That matches the generated text from F2.
  - The stencil sentence appears in this one file only (grep), so no twin copy was left stale.

Further tool calls:
1. `grep -rln "baseline agent's report with the drift it expected" .` to find a twin or embedded copy of the stencil sentence. My context held only the one file.
