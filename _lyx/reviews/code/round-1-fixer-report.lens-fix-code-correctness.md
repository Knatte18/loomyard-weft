```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens on the F1–F4 change set (`resolveShellPath`, `record`, new test rows, `doc.go`).

Walked inputs through each changed branch:
- `$S/../target`: `resolveShellPath` returns `../$S/../target`, which matches no root and is flagged. The old clamp returned `$S/target`, which was excused, so the F1 row fails with the defect in place.
- `~`, `~/`, `~//x` and `~/r/../..`: results are `~`, `~`, `~/x` and an unmatched `../~/r/../..`. The existing `~/r/../..` row still holds.
- `cd $S && git init repo`, then `git -C repo/../repo`: still resolves to `$S/repo`, so the row still passes.
- `git init ../x`, `git init ..` and `git init`: all refused by `record`, so no caller recorded anything the old code did not. Absolute `/` is still recorded, unchanged from before.
- `git init ../x`, then `git -C ../x commit`: the old code recorded `../x`, so the F2 row fails with the defect in place.

The `record` signature and the callers of `resolveShellPath` (`parseGitInvocation`, the `cd` branch and `record`) are unchanged in shape and still hold.

**Findings**

1. **LOW** — `cluster_test.go`, row "git init -- terminator" (`git init -- /tmp/r`, then `git -C /tmp/r commit -m x`).
   - The row passes with the defect in place.
   - If `firstOperand` lacked the `--` case, `--` would be skipped as an option and `/tmp/r` returned as an ordinary operand, so the row still passes.
   - To pin the terminator, the operand must start with `-`, for example `git init -- -r` then `git -C -r commit -m x`.
   - Without the `--` case, `-r` is skipped, no operand is found, and the commit is flagged.
   - The reviewer's own F3 wording asked for `/tmp/r`, so the fix follows the review, but the row guards nothing.

2. **LOW** — `cluster.go`, comment on `firstMutatingGitCommand` (line 74), and the new `doc.go` clause.
   - Both say a `git init` records nothing when it names "the starting directory or an ancestor".
   - `record` now also refuses every path leaving the starting directory through `..`, such as `../x`, a sibling.
   - The `record` comment says this correctly, but line 74 and the `doc.go` clause understate the bound.
   - A reader would conclude `git init ../x` is recorded, and the F2 row shows it is not.

Row "created path with a trailing slash" also passes with or without the fix, but it guards existing behaviour (`path.Clean` strips the slash), so I'm not reporting it.

Tool calls: none; the context already held every changed file.
