```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1 (set: darn setup rows): `darnPhases` maps exactly Preflight, Drift-Baseline and Drift-Await to setup (`timingspans.go:104-106`), and the doc sentence now lists those three. Other docs that state the darn setup set: none found.
- F2 (set: wording about runs affected by the hand settle): the grep found no twin of the old "between Drift-Baseline and Darn-Write" wording outside `_lym`. The recipe header is the only site.

Calls:
1. Bash: grep of `darnPhases` in `timingspans.go`, plus a repo-wide grep for the old F1 and F2 wordings. My context lacked the table's rows and any twin sentences elsewhere.
