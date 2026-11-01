```yaml
lens: fix-affected-set
outcome: reported
```

Findings F1 and F3 are test additions; their sets (former-value rows) are covered by the two new batcher rows, and no further unmarked/marked variants exist for the single `active` former value.

No findings.

Unchecked: F3 (set is the template's two profiles, both asserted by the new subtest).

Tool calls:
1. `grep -rni cautious` outside tests and `_lym`/`.lym`, filtered for "default": to rerun F2's search; my context lacked a repo-wide result. It found only a research-data row, not a doc claiming cautious is the default.
