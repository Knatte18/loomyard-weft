# Card 12 — delete-half-removed-helpers

**Delete:**
- `internal/fabricengine#Topology.RemovePairBranch`
- `internal/fabricengine#PairBranchResult`
- `internal/fabricengine#PairSiblingRemnant`

**Edit:**
- `internal/fabricengine#PairComplete`
- `internal/fabricengine/pairbranch_archive_integration_test.go`
- `internal/fabricengine/cleanupremote_integration_test.go`
- `internal/fabricengine/doc.go`

**Intent:**
Removes the exported fabric helpers left with no production caller once card 11 dropped batten's half-removed special-casing (D1).
`Remove`'s finish path (card 2) covers what they did.

- Delete `Topology.RemovePairBranch` and `PairBranchResult` from `cleanup.go`, and `PairSiblingRemnant` from `fabric.go`.
- `PairComplete`'s doc comment cites `RequireDrivableWorktree` for the vocabulary reason instead of the deleted `PairSiblingRemnant`.
- In `pairbranch_archive_integration_test.go`, delete the three `TestRemovePairBranch_*` tests and keep the three `TestCleanup_*` tests and `wantArchiveTag`.
  Card 2's finish-path tests already cover archive-from-origin.
- In `cleanupremote_integration_test.go`, delete `TestRemovePairBranch_DeletesLocalAndRemoteAndRefusesALivePair` and the file comment's mention of `RemovePairBranch`.
- In `doc.go`, drop the `PairSiblingRemnant` and `Topology.RemovePairBranch` paragraphs and say instead that `Remove` itself finishes a half-removed pair.

`go build ./...` and `go vet ./...` (all tags) must show no remaining reference.

**ImpactSummary:** Deletes three exported fabricengine symbols whose only production caller card 11 removed; test and doc references are cleaned in the same card.

**Commit:** `12: delete fabric's half-removed-pair helpers`
