# Card 5 — planglyph-delta-floor

**Edit:**
- `internal/planglyph#Delta`
- `internal/planglyph#index.Delta`
- `internal/websterengine#postBatchChecks`

**Uses:**
- `internal/gitexec#CheckFloor`

**Intent:** D4: `Delta` calls `gitexec.CheckFloor()` after `openRepo` and before `repo.DeltaGit`, whose git spawn runs inside quarry's `internal/gitsrc` and never reaches gitexec's exec core.
A floor error returns as `fmt.Errorf("planglyph: delta %s..%s: %w", fromSHA, toSHA, err)`, not folded into `ErrQuarryUnavailable`, since a git below the floor is no quarry outage and its message names the way forward.
So `index.Delta` returns it bare, and `postBatchChecks` stops the batch on it as on any other non-quarry error: the intended plain failure, which in practice the same process's gitexec check has already raised at an earlier git command.
`Delta`'s godoc replaces its sentence that every non-nil error is wrapped with `ErrQuarryUnavailable` with the two outcomes: a quarry failure wrapped with `ErrQuarryUnavailable`, and the floor error returned as itself.
`index.Delta`'s godoc and the comment above `postBatchChecks`' `index.Delta` call name the floor error beside the quarry degrade.
Bound: `internal/gitkit`'s own git spawns stay unchecked; it is test plumbing no `lyx` command reaches.
No test injects a version line here: `planglyph`'s tests cannot reach gitexec's process cache, whose injection seam lives in gitexec's `export_test.go`, so card 4's `TestCheckFloor` pins the check and this call is held by review, as the decision record's acceptance criterion says.

**ImpactSummary:** One added guard call in `Delta`, whose one caller chain (`index.Delta`, then webster's `postBatchChecks`) stops a batch on it as on any non-quarry error; three comments follow, and quarry's spawn is otherwise unchanged.
