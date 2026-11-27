```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the F1 fix, covering whether every site making the plan-rule coverage claim was fixed.

Unchecked findings: none. F1 is the only finding, and I reran its search.

no findings

The search for the gap and plan-rule coverage wording under `_lym` (reviews excluded) found two sites:
- `_lym/landing/summary.md:21`, which the fix changed.
- `_lym/discussion/decision-record.md:29`, the parent's decision ("names … the remaining gap the plan rule covers"). It is the instruction the description answers, not a second copy of the overstated claim.

Tool calls:
- One `grep` over `_lym` (reviews excluded) for the coverage wording. My context's earlier search used different terms and did not show whether another site carried the claim.
