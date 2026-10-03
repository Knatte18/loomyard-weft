# Fixer report — plan round 2

All six findings are fixed.
`lyx loom validate-plan` passes on the edited plan with informational findings only.

F6 was found during job B, while grepping for F4's other "integration bisect" mentions.
It was appended to the review before any fix for it was made, and its block says so.

## What changed per finding

- **F1 (MEDIUM):** card 15 now targets `internal/websterengine/gitwrap.go` (moved from `**Uses:**` to `**Edit:**`).
  A new "commit walk" section names the two helpers it adds there: `commitsSince`, over `commitsBetween`, oldest first, which feeds both the check and the report's fix commits, and `fixCommitRejection`, which returns the first commit that is neither a non-merge commit nor a clean parent merge, with the reason.
  `verifygate.go` reaches both through a seam and never constructs a repo, so `TestNoRawGitMutation_WebsterProductionSource` holds.
  The integration test pins the real `fixCommitRejection`, and the ImpactSummary names the helpers.
- **F2:** card 17 targets `gitwrap.go` and deletes `integrationFixHeadRefusal` and `commitParentCount`.
  It states that `commitsBetween` stays under `commitsSince` and `commitChangedPaths` stays under the card hint.
- **F3:** card 17 targets `cmd/lyx/rawgitmutation_test.go`.
  It rewords the header comment's `FabricBisector`/`FabricResetter` example, and the `gitwrap.go` allowlist reason gains card 15's fix-commit walk.
- **F4:** card 17 targets `internal/fabriccli/merge_verbs.go`, which drops the bisect clause from `websterInFlightWarning` and the merge-in help text.
  Per D2, it also targets `mergein_webster_integration_test.go`, whose `assertOneWebsterWarning` asserts `pre-merge trees`.
- **F5:** card 17 targets `internal/verifyrun/run.go` and `internal/websterengine/recoverbatch.go`.
  Its intent names the three stale comments: `run.go`'s package doc, `recoverbatch.go`'s CardSHAs comment and `state.go`'s `CardSHAs` field doc.
- **F6:** new card 20, `retire-detach-primitives`, deletes `CheckoutDetached`/`RestoreBranch` from `gitrepo` and `fabricengine` along with their tests.
  It narrows `gitrepoboundary_test.go`'s pinned method set and rewrites every doc passage that justified the pair by the integration bisect.
  It `**Uses:**` `internal/websterengine/triage.go`, so the derived edge orders it after card 17.
  The Card Index and the overview's opening paragraph name it.
  It was appended as card 20 rather than inserted after card 17, to avoid renumbering cards 18 and 19 and their cross-references.

## Prior rounds

Round 1's fixes hold: card 3 still targets the driver-bootstrap test, and the 14/15/16 split, the exported gate constructors and the parity test are intact.
`accumulatedCardSHAs` survives card 17.
Round 1 deferred nothing.

## Deferred

None.
