---
round: 1
focus:
  - "internal/fabricengine/livestate_verbs_test.go and livestate_matrix_test.go: does each registered Build avoid os.Setenv, os.MkdirTemp, git config --global, a retained TB Context(), package-global seam setters and once-only registration, per D7's review criterion?"
  - "livestate_verbs_test.go: does every Build moved out of an Arrange leave the fixture value unchanged, and does every arrange that places a directory outside the hub, or leaves the board HEAD unequal to its upstream, stay whole in the test?"
  - "livestate_matrix_test.go: is each template registered once per verb and anchor, and do the Add, Pull, Reconcile, hostile-input and CloneHub{Reset} cases still add their own pairs as D7 requires?"
  - "mergein_integration_test.go and the other merge*_integration_test.go files: do the newMergeLayerFixture layers reproduce the replaced commit steps exactly, and do only layers whose branch commits repeat at least three times move?"
  - "Merge integration tests that install a process-global seam: do they take their copies before installing it, per D7's review criterion?"
  - "weftguards_integration_test.go and mergein_recovery_integration_test.go: do the removed lines drop any assertion, or any step that cannot run in a build?"
  - "Scope: does the diff touch anything beyond internal/fabricengine, and does it move sites D7 leaves in the test, such as per-row slugs chosen by a test?"
  - "Commit messages: do the claims about which cases and layers moved match the diff?"
---

Seeded from the decision record's D7 acceptance criteria and the 11-file diff, all under `internal/fabricengine`.
