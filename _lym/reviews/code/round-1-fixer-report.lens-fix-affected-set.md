```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1 (set: the helpers and branches with no test):** the search shows `keyListWayForward` and `keysOutside` are the only helpers the diff added. `keysOutside` is called only from `keyListWayForward`, and `keyListWayForward` is called at three sites in `TestModules_EveryTemplateKeyClassified`. `TestKeyListWayForward` has a row for each branch the function has: add only, drop only, add and drop together, and order only. The expected strings match the code's output formats.
- **F2 (set: the doc-comment sites):** the only comment with the garbled phrase was at `configreg_test.go:202`, and it is now reworded.

Tool calls used:
1. `grep -rn "keyListWayForward\|keysOutside"` over the Go files. My context had no repo-wide caller list for the two helpers.
