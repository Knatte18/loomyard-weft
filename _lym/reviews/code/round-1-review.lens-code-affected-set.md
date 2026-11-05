```yaml
lens: code-affected-set
outcome: reported
```

Task: affected-set lens. For each change the diff makes, I looked for everything outside the touched lines that calls, reads or asserts it.

**Per change: the set found, and whether it is covered**

1. **`landDeps` gains `Tasks`, `FileTask` and `CodeReviewCaps`.** The only constructors are `deps.go:79` and the test fake at `land_test.go:76`, and both are updated. Covered.
2. **New verb `merge-parent`.**
   - The cobra tree, the `cli.go` file header, the `doc.go` package doc, `docs/overview.md` and the `onHubLandDeps` comment are all updated. Covered.
   - Other places mention `beam reopen`: `contracts/recipes/pawl-recipe.yaml`, `internal/loomcli/landingdeps.go`, `internal/pawlengine/doc.go` and `internal/loomrecipe/doc.go`. They describe reopen's own behavior and don't enumerate the beam verbs, so they need no change. Covered.
3. **Reed beam buttons.**
   - Both templates and `config_test.go` are updated.
   - The `prime` group already lists the other beam buttons without `merge-in` and `land`, so the prime does not fall back to the beam group. Covered.
   - I found no doc that enumerates the default button labels; `buttons.go:62` only states the fallback rule. Covered.
4. **Stencil `loom-rubric-code-review.md`.** `contracts/stencils/rubric_test.go` allows only a fixed set of top-level markers, and the added lines use none. The existing rule is kept: the added lines narrow the range only for a merge task and leave the rest untouched. Covered.
5. **PATTERN entries.**
   - The diff carries no `Pattern-Departure:` trailer, and I found no declared PATTERN changes.
   - `PATTERN-parent-lock` is not touched, because the verb only files a task.
   - `PATTERN-named-constants` is touched; see the finding below.

**Findings**

- **LOW, `consistency`: `internal/beamcli/mergeparent.go`, `openMergeTask`.**
  - It tests done-ness with `task.Status != nil && *task.Status == "done"`, a repeated literal.
  - `boardengine.IsDoneStatus` and `StatusDone` exist at `internal/boardengine/store.go:454-458`, and `PATTERN-named-constants` says to use one named constant.
  - Fix: call `boardengine.IsDoneStatus(task.Status)`.
  - `mergeparent_test.go:20` repeats the literal `done := "done"`, which the same fix covers.

I found nothing else the diff leaves wrong.

**Tool calls**
1. A grep for `landDeps{` constructors, for files mentioning `beam reopen` and for the board's done-status constants, to find the full caller set and the named constant.
2. A grep for other docs and code that list the orch and button verbs, and for rubric and merge markers in contracts and internal, to find the docs and stencil tests that might need to follow the change.
3. A grep of docs, the `reedengine` button code and contracts for default-button enumerations, and of `rubric_test.go` for the allowed markers, to check whether anything pins the button labels or the rubric text.
