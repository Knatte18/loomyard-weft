```yaml
round: 1
review: _lym/reviews/code/round-1-review.md
gate: lym gate test --tier 1 over internal/pairteardown, internal/battencli, internal/fabricengine and internal/fabriccli, passed after the last change
```

## Changes

Each fix is its own commit; the self-check's fixes follow the six findings.

- **F1** (87221e29, 94178bf7): fixed as proposed, by amending the pattern.
  `pattern/PATTERN-fabric-git.md` and the destruction-chokepoint section of `internal/fabricengine/doc.go` now name `Topology.RemoveStrayTaskPath` as a second exception beside `removeWorktreeLeftover`.
  The text says the caller proves ownership, that the function refuses a registered linked worktree and a directory holding a `.git` entry, and that `--force` and dirtiness do not apply.
  The `removeWorktreeLeftover` doc comment is back to its narrow wording, and `RemoveStrayTaskPath`'s doc says containment comes from the `os.Root` delete and no dirtiness or force applies.
  Both commits carry a `Pattern-Departure: PATTERN-fabric-git` trailer.
  The second commit rewords the ownership proof after the self-check: a run that created the pair at the path, or an operator who asked to remove the slug's pair, since `cancel --remove` sets `ClearStray` on the operator's request.
  The `ClearStray` comment in `internal/pairteardown/teardown.go` says the same.
- **F2** (912f71f3): fixed as proposed.
  `TestTeardownRequest_ClearsStrayAndLandsTheRemovalRemotely` in `internal/battencli/wire_test.go` asserts `ClearStray`, `Remote` and `Landed` are true and `Force` is false.
- **F3** (095562cc): fixed as proposed.
  A `TestPairParts` row plants a plain directory at a slug's task worktree path and asserts `PairParts{StrayPath: true}` and `Any()`.
  The registered-worktree side is the existing `AfterAddEveryPartIsPresent` row, whose expected value has `StrayPath` false.
- **F4** (ce1c84cb): fixed with a changed placement.
  No `beamMembers` test existed, so the subtest `MemberHoldsAPairOnlyThroughARegisteredWorktree` in `internal/battencli/beamland_integration_test.go` plants a plain directory at member `t` and asserts `HasPair` false, and asserts `HasPair` true for the registered member `c`.
- **F5** (88f9c124): partly fixed, one part deferred.
  `TestRemoveStrayTaskPath` gained rows for an invalid slug and the prime's slug, each asserting an error and that the directory is kept.
  `TestRemove_ReturnsTheRemoteStepFailureOverAKeptDirectory` asserts a remote step's error wins over a kept directory and `StrayPath` still names it.
  The deferred part is below.
- **F6** (31e251d0, 2d8120d6): fixed as proposed, through a different report shape than the `Warnings` list.
  `RemoveResult.StrayPathDeleted` is set by the teardown's stray delete, `cancelReport.StrayDirectoryDeleted` copies it, and the cancel envelope and its help text carry `stray_directory_deleted`.
  Rows in `TestRemovePair_ClearStray` and `TestCancelRun` pin it.
  `lym fabric remove` maps `stray_path_deleted` too, with rows in `removefields_test.go`, and `internal/pairteardown/doc.go` says `RemovePair` sets it.

## Gate repair

The verify gate failed `TestCLIForm_EveryCommand`: the `lym batten cancel` help text was 2013 bytes against a cap of 2000, after F6 added `stray_directory_deleted` to its Output list.
I shortened two phrases ("each deleted tip" to "each tip", "spawns the push of the records" to "pushes the records"), which brings it to 1993 bytes, and committed it as "battencli: fix gate".
`lym gate test` over `cmd/lym` and `internal/battencli` passes at tier 0.

## Self-check

Four forks returned reports, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-consistency, LOW: `internal/pairteardown/doc.go` never mentioned `StrayPathDeleted`. Fixed as proposed.
- fix-consistency, LOW: `fabric remove`'s field map did not map `StrayPathDeleted`. Fixed as proposed, with rows.
- fix-proposal, MEDIUM: `cancel --remove` does not know a pair stood at the path, so "the caller knows a pair stood there" was false. Fixed with my own change: the ownership wording now names the operator's request as a second proof, instead of changing cancel to refuse.
- fix-code-correctness, LOW: dropping the respawn skip in `remove.go` leaves a spurious warning on the `ClearStray` flow. Fixed as proposed: the skip is restored.
- fix-code-correctness, LOW: a refilled directory that `Remove` itself deletes sets neither `StrayPath` nor `StrayPathDeleted`. Left as it was; that is the pair's own late-writer cleanup and out of F6's scope, and the lens called it arguable.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- **F5, the respawn-skip assertion.**
  The review asked for an assertion that no pair push was respawned after a kept directory.
  A mutation run with the skip removed passed the proposed assertion: a plain directory resolves to no pair, so `respawnPairPush` spawns nothing either way, and the skip's only effect is a warning log line.
  `fabricengine` has no logger capture seam, and adding a production seam for one log line is out of proportion, so the skip stays and has no test.
  I restored the skip after the self-check showed removing it adds that warning.
