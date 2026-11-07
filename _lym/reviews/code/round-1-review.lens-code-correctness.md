```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness, for the clone-ordering diff.

**Per function and branch (case run, verdict)**

- `CloneAndWire`, new call site (`internal/fabriccli/clone.go:135`), with a seam installed and failing: `res.SeamWarning` is set, the walk succeeds and `return res, nil` carries it. Correct.
- Same site, no seam installed: `NotifyWorktreeMade` returns "" and `seam_warning` is "". Correct.
- Same site, ordering against the Goal's first suspect (render reads config before the module configs are committed): not settled. See F1.
- `CloneAndWire`, walk failure (`internal/fabriccli/clone.go:144-146`) after a failed render: `return fabricengine.CloneResult{}, err` wipes `res`, and `runCloneWithReset` takes the `errWithRecord` path, which has no `seam_warning` field. The warning survives only in the log. Not correct against "a failed render is printed". See F2.
- `CloneAndWire`, error returns before the call site (lines 47–128): the seam has not fired and `SeamWarning` is empty. Nothing is lost, but the prime is unrendered when `ReconcileAll` or the config commit fails. This matches the old behaviour only for failures after the old call site.
- `runCloneWithReset`, success: `"seam_warning": res.SeamWarning` is always present, which matches the "every field is always present" doc line. Correct.
- `fabricengine.CloneHub` (`internal/fabricengine/clone.go:413`): its `CloneResult` literal leaves `SeamWarning` empty, as the field comment says. Correct.
- `RegisterTeardown` rename: `hub.go`, `template.go` and `hub_test.go` are all updated in the diff. Correct. This is a test-support export in a non-`_test` file; I raise no finding because the Goal's scope is another lens's job.
- New subtest (`worktreeseam_integration_test.go:206-220`): the clone succeeds with a failing seam, and the warning names `res.PrimeCwd` and `lym heddle render`. `failWithin(container)` matches `event.Root` under the container. Correct. The earlier subtests share `seam`, so the new subtest's events do not disturb them: the other assertions use `seam.under(hub.Path, …)`, and the container is a separate temp dir.

**Findings**

- **F1 — MEDIUM, design.** `internal/fabriccli/clone.go:131-135`. The move to after the module-config commit probably does not change what the render reads.
  - The heddle render loads its config only through `configengine.Dirs{Board: boltDir}` and `LoadOrTemplate` (`internal/heddleengine/shipped.go:107`, `config.go:49-50`). The per-worktree module configs that `ReconcileAll` writes and the commit records are not an input, and an absent config falls back to the template.
  - The board's repo defaults are seeded later, by `hubreconcile.Ensure` (`clone.go:142-146`), which is after the event.
  - The diff therefore reorders without confirming the cause the Goal calls unconfirmed.
  - The only demonstrated change is the new `seam_warning` output. No test shows a clone that previously left the prime unrendered now rendering it. The success path rendering the prime has no assertion.
  - Fix: either fire the event after the walk, which seeds the board defaults the render could read, or add a test that a clone with the real `heddleengine.Seam` leaves the rendered files in the prime.

- **F2 — MEDIUM, design.** `internal/fabriccli/clone.go:144-146` and `:133`. A walk failure after a failed render drops `SeamWarning`.
  - The `errWithRecord` envelope carries no `seam_warning`, so the operator is never told, which is exactly what the Goal wants to stop. The comment at `:133` claims a walk failure still leaves the prime rendered, which is false when the render failed.
  - Fix: on the walk error, append the warning to the returned error text (for example, wrap it with `res.SeamWarning` when non-empty). Alternatively, fire the event after the walk, which removes the case. Correct the `:133` comment.

**Tool calls**
1. `grep` of `internal/heddleengine` for `configengine`, `ConfigFile` and `Load(`, plus the `InstallWorktreeSeam` callers. My context lacked what the render reads, which F1 depends on.
