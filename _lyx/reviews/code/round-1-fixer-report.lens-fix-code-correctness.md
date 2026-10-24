```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

The change is one fixture string. The new ID `PATTERN-old-hub-rule` has the same `PATTERN-<kebab>` shape as the old one. The line still carries a `changes:` action, so the row still exercises that action. Nothing in `internal/discussionparser/` asserts on or references the old or new ID, so no assertion or caller depends on the renamed string. The gate run of that package passed in the parent's context. No signature or behavior changed, and the fix added no test.

Tool calls:
1. `grep -rn "hub-suffix\|old-hub-rule" internal/discussionparser/; git show --stat HEAD`: checked that no other reference to either ID exists in the package and that the commit touches only this one line. My context held the edit but not the rest of the package or the commit's scope.
