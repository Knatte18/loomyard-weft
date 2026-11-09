```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked set: F1 (doc comments displaced in `verbs_test.go`).

- The `func step*` grep shows `stepProjectRelease` (547-548) and `stepProjects` (567-568) each have their own doc comment, so the fix covers both.
- The `step*` functions without a doc comment (e.g. `stepPromote`, `stepPrune`, `stepFind`) have a blank line above them, so they never had one to lose. They are not a displacement.
- F2 and F3 are coverage gaps on single branches, not sets, so there was nothing to re-search.

Tool calls:
1. `grep -n -B1 "^func step" internal/boardcli/verbs_test.go`: my context did not hold the doc-comment-to-func adjacency of the other `step*` functions, which F1's set premise needed.
