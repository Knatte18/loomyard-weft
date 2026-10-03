---
round: 1
focus:
  - "Comment convention: CONSTRAINTS.md's `Comment Line-Break Convention` requires semantic line breaks in new and changed Go comments. Do the doc comments card 10 adds or changes in `internal/loomcli/selfreport.go` (`detectAndFileAnomalies`' new paragraph, `detectFinalAnomalies`, `runFilingPass`, `pendingAnomaly`, `selfreportFiledMarker`) and the new test doc comments in `selfreport_test.go` and `selfreport_github_test.go` follow it, or are they wrapped at a fixed column? Untouched legacy lines are out of scope."
  - "`runFilingPass` in `internal/loomcli/selfreport.go`: settle whether a title stays in at most one of `Titles` and `Pending` on every path (retry success, retry failure, a pending title detected again in the same pass). Settle whether each `writeFiledMarker` persists the in-memory marker without losing an entry written earlier in the pass, and whether the early return now taken after the marker read is still correct when the marker is unreadable."
  - "Step entry observation in `internal/loomcli/arm.go`: settle whether `observeStepEntry` at the end of `loomPreStepUnmarked` is reached only on the success path, whether `observeEntry` consumes the step clean-handoff marker the way card 11 states, and whether the observation runs after the ownership check, as `loomPreRun`'s does."
  - "Filing seam in `internal/loomcli/cli.go`: does every `loomCLI` that can reach `loomAfterStep` or `loomPostRun` with `cfg.Selfreport` on carry a non-nil `fileIssue`? Check the `&loomCLI{...}` literals in `arm_seed_test.go`, `parentreview_integration_test.go` and `wiring_commitstatus_test.go`, and any production construction outside `newLoomCLI`."
  - "Tests against card 10's and card 11's intent: does each listed case exist and assert the stated behaviour? This covers the second step of the same halt filing nothing new, the envelope's `friction` reading `reflected`, `TestStepCmd_BusyRefusal_BeforeBootstrap` asserting the marker left in place and `entryObservation` zero, and the github test asserting the pending body. Could any of them pass vacuously, for example because `refuseFileIssue` silently turns a filing into a pending entry?"
  - "Per-card mechanical check: for each card's `Edit` group, confirm the impact check covered every caller of each changed symbol (`runFilingPass`, `detectAndFileAnomalies`, `haltNote`/`writeHaltNote`, `reflectHalt`, `loomPostRun`, `newLoomCLI`). Confirm the plan's `verify:` commands (build, test, integration-tagged test, smoke-tagged vet) pass on HEAD."
---

The review range is `git log --first-parent --no-merges -p 5ee34d63e..HEAD`.
Rework round 1's `first_card` is 10, which matches the plan's, so the range starts at that round's `head_sha` and covers the commits for cards 10, 11 and 12.

Already checked during seeding and not promoted to a focus entry:

- `shedverbs` does not call `AfterStep` when `PreStep` or `BuildShed` refuses.
- The halt-note line is worded the same in `halt.go`, the reflection stencil and the `claims_test.go` claim.
