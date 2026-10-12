# Round 3 fixer report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 16 lists `internal/hubreconcile_test#newStaleHub` under `Edit:`.
  Its Intent says the fixture, after `CopyHub` and `AddPair`, writes and commits a full pre-migration per-worktree file set in the prime and in the pair, each module's template plus one key differing from the repo default, and deletes and commits `configengine.SparseMarkerRel()` in each.
  `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` already builds its hub through `newStaleHub` (`internal/hubreconcile/ensure_integration_test.go:322`), and the Intent now says it keeps doing so, so its restore assertion starts from an unmarked pair.
- **F2** (MEDIUM), fixed as proposed.
  Card 16 lists `internal/hubforge/doc.go`, `internal/fabriccli#CloneAndWire` and `internal/fabriccli_test#TestRunCLI_CleanHubScenario` under `Edit:`.
  A new Intent paragraph says a clone's prime arrives with the sparse marker committed by the clone's walk, that `CloneAndWire` appends a `KindFileWritten` entry for the marker when it is absent before `Ensure` and present after it, that hubforge's seeding contract says so, and that the scenario asserts the committed marker and its record entry.
  The ImpactSummary names the migrated clone and the unmigrating fixture.
  Card 15's Intent says the `CloneConfigCommit_MutationRecordShape` step stops expecting one prime-side `KindFileWritten` per per-worktree module followed by a weft commit, and expects one `KindFileWritten` per seeded repo default, targeting the module's file under the board dir.

## Disputed

None.

## Deferred

None.
