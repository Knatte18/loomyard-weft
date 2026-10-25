# Batch 06: tests card 6 did not list broke on the new clone shape

Clone no longer writes a Bolt marker and no longer leaves the adopted weft tip untouched, so several tests outside card 6's target list failed in the tier 1 run that only the batch gate makes (the card gate is tier 0).
They were fixed under the sanctioned self-fix: `TestRunCLI_CleanHubScenario`, `TestCloneHubReset`, `TestCloneHub_NonEmptyWeftRemoteBranchUnchanged` and `TestCloneHub_AdoptsExistingRemoteWeftPrimaryBranch`.

`refuseEmptyAnchorMarker` in `reconcile.go` still reads only the Bolt marker, so `TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot` cannot reach its refusal by truncating the prime's own marker as the card says.
The test truncates the Bolt marker beside it until a later card makes the refusal read the own marker.
