# Discussion relevant list

Both batches recorded `done`, with no warnings.

## Batch 01, discussion-spec-relevant

Head `987ea2e4f0199f3a40bfb310c85ce91519a4e28c`.
It covers cards 1 to 4: the discussion spec pins `relevant.md`'s format, `DiscussionRelevant` declares its path, `CheckRelevant` checks a present list, and the loom discussion gate and validate-discussion run that check through `ValidateLoomDiscussion`.
The scope guard listed deviations, which are files changed outside the batch's own targets: `internal/discussionparser/relevant.go`, `internal/loomcli/validate_test.go` and `internal/shedrecipe/fixture_test.go`.

## Batch 05, twill-relaunch-files

Head `88fa3ced15fe4dfe07307a9f4e46523c98ebcd5c`.
It covers cards 5 to 7: a relaunched Discussion writer finishes all three files, the Discussion-Write lead lists and writes `relevant.md`, and Plan-Write reads `relevant.md` first when it exists.
The scope guard reported no deviations.

## Verify gate fixes

The gate's first attempt failed on `TestWire_DiscussionTableEvaluatesToExpectedShape`, both the Interactive and Autonomous subtests.
The test still expected the discussion chair's outputs to be the decision record and support log, but batch 05 made the chair write `relevant.md` too.
The fixer added `loomengine.DiscussionRelevant(loc)` to the expected outputs in `internal/loomcli/wiring_test.go`, in commit `c5047ed3e` (`fix: wiring test expects relevant.md among the discussion chair's outputs`).
The fixer ran `lyx gate test --tags integration ./internal/loomcli`, which passed, and reports a clean worktree; it did not run the plan-level verify.
