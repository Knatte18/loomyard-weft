```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's change set through lens fix-code-correctness, run the changed code on concrete inputs and check that each added test fails with its defect in place.

No BLOCKING or MEDIUM findings.

**LOW: `holdsCall` treats any call expression as effectful.**
- Changed text: `internal/testcover/mutant.go`, `holdsCall`, used in `logsOnly`.
- It also matches `len(x)`, `cap(x)` and type conversions such as `int(x)`.
- An `if len(items) > 0 { logger.Info(...) }` that was log-only noise before now gets mutants again.
- This is the conservative direction the review's fix asked for ("holds no call expression"), so it is a noted trade-off, not a defect. Nothing in `linemutant_test.go` pins it.

**Walk-throughs, all consistent with the code and with the tier 0 and `TestGateMutate` runs that passed:**
- **`mutationStep.recordFindings`:**
  - Failed call (err = slot refusal): it appends the refresh error and the call's error. It copies the partial counts, then returns before `Functions` and `FileHashes`.
  - Cancelled call: `reach.ctx.Err() != nil`, so the error becomes `reasonCancelled`.
  - Success: functions are mapped and `goFileHashes` fills `FileHashes`; a hash failure is appended and the loop continues.
  - `append(found.notJudged, …)` can alias `found.notJudged`, but it is nil in the shared test fixture, so the parallel subtests do not race.
  - The old `answer` returned `Functions` and `FileHashes` on error. Restoring that fails row 1 of `TestMutationStep_RecordFindings`, since `Functions` would be non-nil.
- **`Config.HasGoCoverageMaps`:** an unknown or empty scoper returns false, the same as the old `!= "go"`. The scoper row in `TestGateMutate` sets `cfg.Scoper = "none"` and takes the note branch.
- **`logsOnly` with `holdsCall`:** with the call check removed, the outer if of `Counted` (line 107) becomes noise and loses its mutant, so that row fails.
- **New `TestPlanLineMutants` rows:** each fails if the code it targets is broken.
  - Else and else-if rows (76, 81, 83): breaking the `default` or `BlockStmt` arm of `logsOnly` gives line 81 or 76 a mutant.
  - Init row (89): removing the `Init` refusal drops the mutant.
  - Reversed-literal row (95): breaking the operand swap gives the comparison a mutant.
  - Shadowed-`runtime` row (99): removing the locals check drops the mutant.
- **`MutationReportMatches` early return:** a nil location is safe in `TestMutationReportMatches_RefusesAReportThatJudgedNothing` because the note, red-gate and prime guard returns before any use of it.
- **`TestGateMutate` red-gate row:** removing the `red` case in `run` makes `answer()` run a real check, leaving `Note` empty, so the row fails.
- **`TestGateMutate` no-parent row:** it runs last, so removing the pair's origin record affects no later row.
- **F6 row:** the stale `report` was written for the earlier HEAD, and Clamp's hash changes with the new commit, so the match is false.

**Out of scope, one line:** the red-gate row calls `step.run(true, true)` directly. It does not exercise the exit-code-unchanged wiring from the call path, which the reach tests do not exercise either.

Tool calls: none; my context already held every changed file and test.
