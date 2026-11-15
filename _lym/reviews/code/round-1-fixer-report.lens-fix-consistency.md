```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set against its surroundings for stale twins and cross-reference disagreements (lens fix-consistency).

**Findings**

1. **MEDIUM — `internal/beamcli/create.go:158` and `internal/beamcli/cli.go:786` still describe the old done-run refusal.**
   - `create.go:158` says "Any other done run stays, and the start refuses it."
   - `cli.go:786` says the run "is still refused, naming `lym batten cleanup --apply`."
   - F2's fix updated `internal/beamcli/doc.go` and `docs/overview.md` but not these two.
   - Both are twins of the `doc.go:59` text the fix changed, and the dobby's clear now makes them stale in the same way.
   - `cli.go:786` is user-facing help for `lym beam create`.

2. **LOW — `internal/dobbyengine/donerun.go` header and `clearDoneRuns` doc still describe records only.**
   - The header reads "the removal of a finished run's records from the prime."
   - The doc on `clearDoneRuns` reads "removes, through Deps.ClearDoneRun, the records of each listed run."
   - The twin edits in `loop.go` (`Deps.ClearDoneRun`) and `doc.go` now say records, scratch and task branch.
   - The `clearDoneRuns` doc also omits the failure case where a failed branch removal keeps the records.

3. **LOW — The `internal/battencli/cleanup.go` file header omits the branch.**
   - Lines 3–4 say `ClearDoneRun` and `ClearMotifRun` "remove one slug's done run."
   - Since `ClearDoneRun` now also removes the task branch, this is a minor stale detail, though still accurate.

**Checked, consistent**
- `doneSlugRefusal` in `internal/battencli/arm.go`: the review asked to change it only if the branch stays, and the branch now goes, so the manual-path text stays accurate until the dobby clears the run.
- The `ClearDoneRun` call in `dobbydeps.go` and the `clearDoneRun` signature: no other callers beyond `ClearMotifRun` and the tests, all updated.
- `battencli/cleanup.go` `cleanupCmd` help: it already mentions the dobby's clear and makes no branch claim.

**Calls made**
1. `grep -rnE "cleanup --apply|ClearDoneRun|done run (stays|is refused)"` over `internal` and `docs`: to find twins of the changed text outside the files the fix touched, which my context did not show.
