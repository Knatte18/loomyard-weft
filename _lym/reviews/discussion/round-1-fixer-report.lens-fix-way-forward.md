```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix added can be carried out and leads to progress.

Ways forward the fix added:
1. D1 `decision-heading-malformed`: rewrite the heading as `### D<n> — <title>`.
2. D1 `decision-id-duplicate`: renumber one of the decisions and the acceptance bullets that name it.
3. D2 ordering: one re-prompt names every defect, and the darn history check runs only on a clean record.
4. D3, D6 and D8 Bound escape hatches, and D8 test/non-test classification through the plan gate's index.

Findings:

- **LOW.** D1, `decision-id-duplicate` way forward ("renumber one of the decisions and the acceptance bullets that name it").
  Renumbering also has to update every other `D<n>` mention, but the way forward names only acceptance bullets.
  D2 checks only acceptance bullets, so a stale mention elsewhere passes silently and now points at the wrong decision.
  Evidence: this record cites ids outside acceptance bullets in Scope ("(D1, D2)", "(D5, D6)"), D4 ("as D3 says"), Constraints ("(D10)", "D3 to D8") and Auto-mode assumptions ("(D4)", "(D5)", "(D6)").
  The way forward should say "and every mention of its id".
- **LOW.** D1, `decision-heading-malformed` way forward ("rewrite the heading as `### D<n> — <title>`").
  This assumes every stray H3 under `## Decisions` is a decision missing its id.
  When the H3 is a sub-heading inside a decision, such as a `### Rationale`, following this creates a fake decision.
  D2 then demands an acceptance bullet for it, so the re-prompt leads to a worse record.
  The way forward should also offer: demote it to H4 or plain text when it is not a decision.

No finding on the D8 classification path.
Both builders already hand `c.env.PlanIndex` and `env`'s index to `ValidatePlan`, `ValidateReworkPlan` and `ValidatePlanLensReview` beside `pc` (`internal/loomcli/validate.go:283-289`).
A classification lookup's quarry error goes out on the existing `ErrQuarryUnavailable` envelope, so the step is reachable.

Tool calls:
- Read `internal/loomcli/validate.go:255-300` and `internal/shedrecipe/entries_gate.go:115-140`, to confirm the index and `RefuseRaw` reach both `PlanContext` builders. My context showed neither builder.
