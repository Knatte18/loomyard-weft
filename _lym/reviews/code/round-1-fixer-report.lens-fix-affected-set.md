```yaml
lens: fix-affected-set
outcome: reported
```

No findings. Two sibling-text gaps are below as LOW; each is a stale doc line, not a missed site.

Searches by finding:

- **F8 (parse sites):** every other `parser.ParseFile` in `internal/testcover` and `internal/gatecli` already passes `SkipObjectResolution`.
  The one in `mutant.go:377` now does too.
- **F3 (composite-key text):** the only text still stating the old rule is `mutant.go:502`, which is already updated.
- **F6 (delete-clause text):** `internal/testcover/doc.go:45` is the package doc for the operator.
  It does not say the operand is still evaluated.
  The fix put that statement only in the `OperatorDeleteClause` comment and the `PlanMutants` doc.
- **F5 (variable-attribution text):** `Mutant.Blocks` at `mutant.go:123` still says "each block that reads the variable".
  `internal/testcover/doc.go:47` still says "each block that names the variable".
  Both are now stale, because a variable that only another variable's value names also gets its reader blocks, which F5 added.

**LOW: `internal/testcover/mutant.go:123`, `Mutant.Blocks` doc.**
- The doc says "a site of a package-level variable's value has each block that reads the variable".
- The fix to `PlanMutants` (`mutant.go:205`) widened this to include blocks that read a variable whose value names it.
- The sibling field doc was left as it was.

**LOW: `internal/testcover/doc.go:47`, package doc.**
- The doc says "A literal of a package-level variable's value is a site of each block that names the variable."
- This is the same widening, left unstated in the package doc.
- The delete-clause line at `doc.go:45` also lacks the F6 caveat that the operand is still evaluated.

Tool calls:
1. Bash grep for `parser.ParseFile` and `SkipObjectResolution`: the F8 set was not in my context.
2. Bash grep for "composite key", "must be constant" and "map entry": the F3 doc set was not in my context.
3. Bash grep for "delete clause", `OperatorDeleteClause` and "names the variable": the F5 and F6 doc set was not in my context.
4. Read `internal/testcover/doc.go` lines 36–59: the file was not in my context.
