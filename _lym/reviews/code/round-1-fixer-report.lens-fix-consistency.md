```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for stale siblings and unmatched twins (lens fix-consistency).

**Findings**

1. LOW. `internal/testcover/mutant.go:123`, the `Mutant.Blocks` field comment.
   - It says a package-variable site "has each block that reads the variable".
   - The `PlanMutants` doc at `mutant.go:205` now adds "and to each block that names a variable whose value names it".
     F5's propagation makes the field comment too narrow.
   - Evidence: `grep` shows the old wording at line 123 and the new wording at line 205.

2. LOW. `internal/testcover/mutant.go:43`, the operator constants group.
   - The comment added above `OperatorDeleteClause` documents one constant in a group where its siblings carry none.
   - It is accurate.
     A reader may still notice that `OperatorChangeEmbeddedLine` and the other operators have no comparable per-item comment.
   - Say so if you consider this too minor to act on.

No other stale text found.
`grep` over the repo's `.go` and `.md` files (outside `_lym`, `.lym` and the tests) shows no other mention of composite keys, delete clause or `ParseComments` that needed a matching edit.

The integration test's header comment, mutant and run counts, line numbers and list orders agree with the passing tier-1 run.
That run was 288 passed, 0 failed.

**Calls made**
1. `grep -rn` over `.go` and `.md` files for "composite key", "map entry", "delete clause", "names the variable" and "SkipObjectResolution".
   My context did not show whether docs elsewhere in the repo repeat the changed wording.
