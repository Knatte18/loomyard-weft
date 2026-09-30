---
round: 1
ledger:
  - key: replacestrand-post-add-save-orphans-script
    rounds: [1]
    status: resolved
  - key: addstrand-save-failure-delete-untested
    rounds: [1]
    status: resolved
  - key: emptycmd-integration-test-doc-stale
    rounds: [1]
    status: resolved
  - key: card3-replacestrand-test-launch-unspecified
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so there was no prior ledger.
The reviewer raised four findings, all fixed in the same round:

- `ReplaceStrand` could orphan the new strand's script when both its post-add save and the reconcile tail's save failed.
  Card 4 now deletes it on exactly that condition.
- The `AddStrand` save-failure delete had no test.
  Card 4 adds one.
- `emptycmd_integration_test.go`'s file doc would have gone stale.
  Card 4 rewords it.
- Card 3's `ReplaceStrand` test did not say how the fake survives the replacement launch.
  It now uses an `AnchorHidden` replacement.

I verified all four fixes against the code, and none carries forward open.
