# PR-Rework starts a new plan generation

All sixteen batches reported `done`, and the plan-level integration suite reported `OK` at HEAD `2c08d0aa2`.

Cards 1 to 4 were already done when this session started: the `first_card` overview key, the Plan-Review exempt classifier, the archive of webster's run record into a told directory, and the session factory that tells the PR-Rework producer its values.

This session ran cards 5 to 16:

- **05 rework-archive-generation:** PR-Rework archives the live generation (plan, webster run record, review run directories) into the round directory and the session writes a whole new plan.
  The fork added `reworkCommitPathspec` and the exported helpers `LatestArchivedReviewsDir` and `LatestArchivedWebsterDir`, which the card did not name.
- **06 retire-plan-fingerprint-rebaseline:** `RebaselinePlanFingerprint` and its test file were removed, and `TestRun_FreshRunOverNewGenerationAfterArchive` pins the fresh-run behaviour.
- **07 rework-gate-whole-plan:** `planparser.CheckFirstCard` and `planglyph.ValidateRework` check the whole new plan against the told first card.
  Deviation: `internal/loomcli/parity_test.go` and `internal/loomcli/validate_test.go` were edited outside the declared targets, because the old rework parity fixtures broke under whole-plan semantics.
- **08 bouncer-skip-seam:** `BouncerConfig.Skip` approves without a judge when it returns true.
- **09 plan-review-skip-predicate:** `loomshed.PlanReviewSkippable` decides from committed state whether the live generation skips Plan-Review.
- **10 rework-routes-through-plan-review:** `PR-Rework.on_done` now routes to `Plan-Bouncer`, which carries `skip_seam: rework-exempt`.
- **11 describe-every-generation:** Describe is told every generation's run record, oldest first.
- **12 plan-review-rework-answer-key:** the Plan-Review rubric and recipe add the live generation's findings to the answer key.
- **13 webster-review-rework-range:** the Webster-Review rubric reviews only the live generation's own commits, with an integration test over a real repository.
- **14 attack-surface-question:** both rubrics and both writer stencils carry the attack-surface question.
- **15 self-contained-focus:** the judge template and focus directive keep a focus entry self-contained.
- **16 status-commit-holds-archive:** `statusCommitPathspec` holds the round back while a rejection is pending.

Several forks reported using a script or shell heredoc to write files, against the no-scripted-rewrite rule.
They described the results as correct.
`gofmt -l` flags `internal/lyxcwd/docslink_test.go`, `internal/shell/posix.go` and `internal/websterengine/strand_test.go`; the forks did not touch them.
