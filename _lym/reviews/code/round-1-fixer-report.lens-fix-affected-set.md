```yaml
lens: fix-affected-set
outcome: reported
```

No findings that need a change; one LOW stale comment.

Findings checked: F1, F2, F3. No finding is left unchecked.

- **F3, the stencil.** Only the Drift-Verify stencil, `drift-template-verify.md:23`, calls `baseline.md` an agent's report with expected drift. The other consumers are the two `BaselineReportPath` inputs in `seats.go` and the baseline stencil, which really is agent-written. The stencil source is embedded from `contracts/stencils`, and no other copy of that text exists. Covered.
- **F2, the provenance claim.** No other text says the drift config set the scope, and the only test assertion on the old `drift.scope` wording was the one the fixer updated. Covered.
- **F1, the guard test.** The new row asserts the seeded report text survives and that agents stay at 0. Covered.
- **LOW:** `internal/driftengine/paths.go:20` still says "`BaselineReport` is the Drift-Baseline agent's report", which is false when the scope is configured. The fix did not touch it, but F3's premise reaches it. Suggested change: say the file is the agent's report, or a generated one when the drift config sets the scope.

Tool calls:
1. Grep for the old provenance and "agent's report" wording across `.go`, `.md` and `.yaml` files, to find every site that makes the claim.
2. Grep for `baseline_path` and `BaselineReport` outside tests, to list every consumer and doc comment about `baseline.md`; my context had none of them.
3. Grep for `drift-template-verify` and `git status --short`, to look for another copy of the stencil and for uncommitted changes. `contracts/stencils/stencils.go` embeds the stencil and there is no second copy, and the tree has no uncommitted changes.
