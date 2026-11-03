```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- The new override bullet in `lym-batching` no longer contains the "the rule above" wording. No other shipped text, test or doc quotes the old bullet. The remaining "the rule above" hits are in unrelated files.
- The numbered sub-steps of step 4 in `lym-plan-edits-before-gate` have no twin. Nothing else restates "Decide every fix" or "Run the gate or validator again".
- `CLAUDE.local.md:5` is the old prose form of the same rule. It is the operator's private file and not generated from `shipped.json`, so I leave it out of scope.
- The two edits are limited to `shipped.json` text. The review's "Never" bullets and the `##` heading structure are unchanged, so the package tests (green per the fixer) are unaffected.

Tool calls:
1. `grep` over the repo for the old and new phrases, to find twins or quoters of the changed text. My context held only `shipped.json`, not the rest of the repo.
