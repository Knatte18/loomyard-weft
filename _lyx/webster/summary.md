# Skills and stencils: review-rejection fixes (cards 17–21)

All five batches reported `done` with no deviations from the declared file-ops; only informational scope warnings on test symbols were returned.

- **17 orch-role-run-loop** (`edf196fc9`): rewrote the run loop, escalation, PR-Gate and after-landing sections of the orch role stencil to match what batten and Finalize do, and added claims rows for it to `wordingClaims`.
- **18 driver-stencil-claims** (`b6fddbd9d`): the driver stencil's phrase checks became a claims row; the recipe-blind scan now also forbids every loom interrupt-policy row name; `internal/loomcli/driverstencil_test.go` was deleted.
- **19 orch-prompt-tests** (`5b7c93538`): the orch stencil wording checks moved into the claims table, leaving one render table test in `internal/orchengine/prompt_test.go`.
- **20 drop-getter-delegation-tests** (`80e7c3df8`): deleted the constant-getter test and the two delegation tests; the skill-load delegation test became a guard-strands test.
- **21 skill-skip-tables** (`78cdf1ec3`): the two skill-skip causes became one table test each in `internal/shuttleengine/startup_test.go` and `internal/orchengine/watcher_test.go`.
