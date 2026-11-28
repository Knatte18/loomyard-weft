```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (fix-code-correctness lens).

**Traced, and holding:**
- **`ownPackageTiers`:** a `cfg.Tier` error, tier0, a tier covering 0, a `carry: false` tier, a `--no-carry` call and a command with no target placeholder are all left out. A narrowable tier1 is still listed.
- **Whole-running tiers:** such a tier now takes `sc.modules` in `listedPackages`. If every candidate runs whole, no `sc.tiers` entry is made and no notice prints. `computeScope` is the only caller.
- **F1 replacements:** each equals the old literal, and the changed packages compiled and passed their tests.
- **F2 assertion:** it fails if `own = withMainPackages(...)` is dropped, because tier1 would run `./lib` alone.
- **F4 assertion:** it fails if `ownSkip` is swapped for `sc.skip`, because tier1 would gain `-skip`. Committing `c0_test.go` on the prime is what lets the watched test be skipped.

**Findings:**

1. **LOW.** Changed text: the F3 condition in `ownPackageTiers` (`internal/gatecli/scope.go`, commit `dfa13325b`).
   - **Summary:** the `computeScope` doc comment was left stating the old rule.
   - **Evidence:** it still says "Under the batch key, each listed tier above tier0 that does not cover tier0 runs the cards' packages alone", and that "request.stderr names those tiers".
   - **Why it is now wrong:** a `carry: false` tier, any tier of a `--no-carry` call, and a tier whose command takes no targets are no longer narrowed or named. Only the `ownPackageTiers` comment says so.

No test fails if the F3 condition is removed, since the fix added none for F3 (a design finding, not a coverage gap). I note this rather than count it as a finding.

Tool calls (1 of 6):
- `git grep` for the `--card` help string plus `git show --stat` of the F3 commit: my context did not show whether another file (a golden or help fixture) copied that string. None does.
