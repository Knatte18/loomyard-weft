# Merge main into the beam adopt-build-resplit

Brings the parent branch `main` into the beam `adopt-build-resplit` as a true merge, with no pull request.
The merge was not up to date, so the beam now carries what `main` gained since the last merge in.

## What the merge brings in

- Slack bridges for orch-to-orch creel mail between machines: `slackengine` bridge files, `creelengine` cross-machine mail,
  `creelcli` bridge wiring, `hubgeom` Slack geometry, `agentname`, and the matching docs, contracts and patterns.
- Codex edit records for the edit-loop watch (`codexengine/editrecord.go`, `shuttleengine/editloop.go`).
- `testcover` mutation of data literals and deletion of boolean-chain operands, with the `gatecli` redundant-mutant guards that follow.
- The hand-over of a run to a new build no longer checks for uncommitted paths (`sheddriver`, `loomcli/drivewiring.go`).

## Fix after the merge

The beam had removed `machinename.Names`, and the Slack bridges that `main` added still call it.
The commit after the merge restores `machinename.Names` and exports `MaxNameLength` as the bridges need.
No other change goes beyond the merge.

## Check by hand

- The gate on the merged tree was not re-run by this description; confirm it passes.
- Confirm the beam's removal of `machinename.Names` was not wanted for any caller other than the restored bridge use.
