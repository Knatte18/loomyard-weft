```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F1 is the only finding. Its premise is a set: tests asserting the literal scratch-ban sentence from the shipped stencils. I searched all `*_test.go` files for `never build, restore or compile`, `scratchBuildBan` and `git worktree, clone or checkout`. There were no matches, so no other test pins that prose, and the removed assertion was the only one.

Findings left unchecked: none.

Tool calls made:
1. `grep -rnE` over the test files for the ban wording and the `scratchBuildBan` identifier. My context held only `editdirective_test.go`, not the rest of the repository's tests.
