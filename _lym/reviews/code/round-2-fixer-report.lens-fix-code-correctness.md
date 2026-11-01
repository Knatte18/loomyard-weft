```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1 change set (commit 8ce1f333, `crucible/README.md` and `crucible/review-prompt-template.md`).

no findings

The commit changes markdown only, with no function, branch, signature or test, so there is nothing to run step by step. The `tiergate.Require(t, N)` form the new text names matches the real signature, `func Require(t testing.TB, tier int)`. That makes the instruction to open a live test with `tiergate.Require(t, 2)` or `tiergate.Require(t, 3)` sound. I did not check whether any test file currently carries that call.

Tool calls:
1. `grep -rn "^func Require" internal/testkit/tiergate/` plus `git show --stat HEAD`: confirmed the `Require` signature and the commit's file set, which my context lacked.
