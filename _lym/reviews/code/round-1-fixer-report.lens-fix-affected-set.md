```yaml
lens: fix-affected-set
outcome: reported
```

The `refscanner.go` classifier is a separate one, and the review explicitly rejected it as out of scope.

No findings.

- F1 covers every site: both maps in `cluster.go`, `doc.go:303`, and rows for `-v`, `--verbose` and `-vv` with an operand plus `-v` alone and `-v -l feat*`. No other copy of the list-mode set exists in the repo.
- F2 is fixed: `--format -x`, `--points-at -x`, `--` and `foo --` rows are present.

Unchecked: nothing.

Tool calls:
1. `grep` for `-v`/`--verbose` mentions in burlerengine, contracts, docs and PATTERN. My context lacked a repo-wide view of other sites that treat `-v` as list mode.
2. `grep` for `gitBranchRead`/`--show-current` repo-wide. I used it to look for a duplicate list-mode classifier.
