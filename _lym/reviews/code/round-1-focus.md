---
round: 1
focus:
  - "Production edits in `dirtiness.go`, `junction.go` and `worktreelist.go`: does each change only what a fold or a tier 0 move needs, with behavior unchanged, as the record's last constraint requires?"
  - "Tier 0 moves (`dirtiness_parse_test.go`, `worktreelist_parse_test.go`, `unwire_exclude_test.go`): does each new test carry every assertion the removed git-backed test made, or does a removed assertion lose coverage?"
  - "Folds (`livestate_states_selftest_test.go`, `livestate_verbs_selftest_test.go`, `livestate_matrix_test.go`, `livestate_states_test.go`): does the folded precondition-field check fail on an empty field, and does the vacuity check run in the fixtures' apply path with the same strength as before?"
  - "Anchor-loop cuts (`scratchlink_integration_test.go`, `weftgit_exclude_test.go`, `TestStates`): do the remaining nested-anchor rows still cover every behavior the dropped rows asserted, including machine-local files kept out of the weft tree?"
  - "Dropped or edited tests in `reconcile_stale_registration_test.go` and `unwire_test.go`: do any assert archive tags and their order, leased or other deletes on origin, user content never claimed or removed, or mutation records and their order, which the record says must stay?"
  - "`//testtiming:keep` marks added across the integration test files: does each carry a reason, and does each test it marks sit in the contained set the record names?"
  - "Commit messages against the diff: does any dropped test lack grounding in the record's rules, such as a rarest-block-count-2 test dropped without its partner?"
---

The record has no plan, so the measuring stick is the `## Goal` entry.
The record asks for a before and after tier 1 timing in the PR; that is not a diff matter.
