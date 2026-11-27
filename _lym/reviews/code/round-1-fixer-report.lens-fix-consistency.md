```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1, F2 and F3 changes against their surroundings.

1. `LOW`: F2's twin in `internal/gatecli/scopes_integration_test.go:66` was left as it was.
   - Changed text: `_lym/landing/summary.md`, `## Design decisions`, bullet "The plan is written for every scopes row", now "It sits under `_lym/plan`, which the pair's code repository excludes".
   - Evidence: the `scopesHub.plan` field comment still reads "plan is the one-card plan written under the pair's never-tracked plan directory."
   - The review's F2 rejects that description, since `_lym` "is the tracked records namespace, not a never-tracked one". The directory is `planparser.PlanDir`, `<anchor>/_lym/plan`. It is excluded only from the pair's code repository.

2. `LOW`: the change description does not cover F1's new setup step.
   - Changed text: row "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor", which now opens with `f.commitChange(t, "other/other.go", …)` before its `--all` run.
   - Evidence: `_lym/landing/summary.md`, bullet "Rows that read an earlier row's state now set it up themselves", names three green-record rows and "a real go test tier in the nested module", but not this row.
   - According to the review's F1, this row relied on the commits an earlier row left behind, and it now sets up that state itself. The bullet's account of such rows is therefore incomplete.

I found no disagreement from F3's comment: it matches the summary's "Rows over a standalone worktree, or outside every hub, build no hub."

I made no tool calls.
