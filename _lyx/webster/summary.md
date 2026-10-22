# machine-handover: owner machine records, checks and `lyx adopt`

All 14 batches ended `done`; head after the batches was `c80788db9`.

- **01 machine-name** (cards 1-3): the `internal/machinename` leaf, `owner_machine` and `prime_owner` records with their replay and effective owner, and `fabricengine.ParentSlug`.
  `record-batch` failed on four test targets that did not resolve; one `recover-batch` cleared it.
  Deviations: `internal/boardengine/owners.go`, `replay.go`, `replay_test.go`.
- **04 board-owner-writes** (cards 4-5): the owner setter, parent resolver and payload rule, and owner in board list, get and the README's Running rows.
  Deviations: `boardcli/verbs_test.go`, `boardengine/{owners.go,ownerset.go,ownerset_integration_test.go,render.go,runlock_test.go}`, `fabricengine/bolt.go`.
- **06 reconcile-owner-outcome** (cards 6-8): the reconcile reports `owned_elsewhere`, pairteardown builds the owner reader, and the owner pause caller label.
- **09 batten-owner-check** (cards 9-10): batten and loom start, resume and claim only what this machine owns.
  `record-batch` failed on the unresolved `TestRunOwnerVerdict`; one `recover-batch` cleared it.
  Deviations: `battencli/{arm.go,lifecycle_integration_test.go,owner.go,owner_integration_test.go,owner_test.go}`, `loomcli/{export_test.go,landingdeps.go,ownercheck.go}`.
- **11 fabric-parent-holds-task-work** (cards 11-13): `ParentHoldsTaskWork`, InnerRun's owner re-check, and tender's owner pause.
  Deviations: `battenshed/{innerrun_owner.go,innerrun_owner_test.go,innerrun_parentlock_test.go}`, `tendercli/{cli_test.go,deps.go}`.
- **14 batten-innerrun-wiring** (cards 14-15): batten's live owner check wiring and `LocalOnlyCommits`.
- **16 finalize-owner-checks** (card 16): Finalize lands only where this machine owns the run and its parent.
  Deviations: `landingshed/{finalize_owner.go,stuck.go}`, `loomcli/{landingdeps.go,wiring_seams_test.go}`.
- **17 wake-and-orch-owner-pause** (card 17): the hibernate wake and orch start pause a run owned elsewhere.
  Deviations: `orchcli/{ownerpause.go,start.go,start_test.go}`.
- **18 tender-status-owners** (cards 18-19): tender status owners and `reedengine.ClientAttached`.
- **20 open-window-on-resume** (card 20): a resume reopens the window when no client is attached.
  Deviation: `battencli/arm.go`.
- **21 fabric-drop-local** (card 21): `DropLocal`.
- **22 batten-resume-in-window** (cards 22-23): `battencli.ResumeInWindow` and `hubforge.CloneSecond`.
- **24 adopt-module** (card 24): the `lyx adopt` module.
  Deviations: `adoptcli/{adopt.go,adopt_integration_test.go,adopt_test.go,deps.go,export_test.go}`.
- **25 orch-role-and-pattern** (card 25): the orch stencil reads a run owned elsewhere as that machine's, plus `PATTERN-owner-machine`.

No digest carried a `departures` list.

## Verify gate findings

The plan-level verify failed on attempt 1 of 3 with these failing identities:

- `cmd/lyx.TestCLIForm_EveryCommand`
- package `internal/battencli`
- package `internal/testkit/envkit`

One fixer fork was spawned for them.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `ec2a3b5b6461105d6d61d836c824c9225569e863` fix: owner seams nil-legal in envkit, board upsert and batten run help under the length cap
