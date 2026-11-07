```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set, rerunning the set searches behind F2 and F3, the findings whose premise is a set of callers or sites.

- **F3, callers of `Seam.render` and `Seam.Made`:**
  - `internal/heddleengine/seam.go:31` (`Made`) and `internal/heddleengine/seam.go:62` (`TookIn`) are the only non-test callers of `render`, and both were adapted to the new `(Report, error)` return.
  - The only test callers of `Made` are `seam_test.go:75` and `seam_test.go:95`. The first still passes, because it writes files. The second is the new warning-only row.
  - No other site uses the changed signature or behaviour, so the callers are covered.
- **F2, sites that fire the made event and drop its warning:**
  - `internal/fabricengine/add.go:723` calls `NotifyWorktreeMade(pair)` and discards the returned warning. The pair-add path therefore still keeps a failed render in the log only.
  - The review's F2 names only the clone path, and the fix covers it.
  - Within `CloneAndWire`, the walk's `hubreconcile.Ensure` error is the only failing return after `NotifyWorktreeMade`. The fix wraps that one. The later `KindFileWritten` loops return nothing.

Findings:

- **LOW** — changed text: `internal/heddleengine/seam.go:31` (`Made` now returns an error on a render that writes nothing).
  - Summary: the pair-add caller at `internal/fabricengine/add.go:723` drops that error's warning, so the F3 symptom survives on that path. A pair made with a warning-only render prints nothing to the operator.
  - Evidence: `NotifyWorktreeMade(pair)` is called as a bare statement, while the clone path assigns its result to `res.SeamWarning`.
  - This is an unreviewed sibling of F3, not a regression, so the fixer can note it as outside the review.

Calls made:
1. Grep for `.render(`, `NotifyWorktreeMade` and `SeamWarning` in non-test Go files, because my context lacked the set of callers and warning consumers.
2. Grep for `.Made(` call sites in test files, because my context lacked the other tests that call `Made` directly.
