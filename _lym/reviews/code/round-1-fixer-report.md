---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "Rows for FetchPairTips and ReturnToPreviousTransition were added as proposed. The FastForwardPair row was not: fastForwardWeft only logs a layout it cannot read, and a records store with no fetch refspec reads as having no upstream, so the weft HEAD stays put with or without the read. No assertion on the code side or the weft HEAD can fail with the read deleted, and the repository has no log capture to assert the warning."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "Both re-runs are named as proposed, but the seed's shortname is always the <shortname> placeholder. The shortname is read from the board directory, which is exactly what the refused hub lacks, so a read of it cannot succeed and would be dead code."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The two-repository table row, the plain_archive_tags empty list and the held-lock row naming a given rerun were added as proposed. The cancel-passes-its-own-command row was not: battencli has no seam to hold a slug's teardown lock or to fail the discard's remote half, and the only hold helper sits in fabricengine's tests. The rerun that reaches DiscardTaskBranch is pinned there by the held-lock row. The cancel call site itself stays unpinned."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Fixer report, round 1

## Changes

Commits, in order:

- `fabriccli: fix F1, F4`: the cleanup help builds its namespace from `fabricengine.RecordsNamespaceRoot`, and `doc.go` spells no namespace.
  A failed `PlainArchiveTagsOnOrigin` sets `plain_archive_tags_error` and the verb goes on.
  `PlainArchiveTagsOnOrigin` treats only `gitrepo.ErrRemoteNotConfigured` as no origin and returns any other `RemoteURL` error.
- `battencli: fix F2`: `fabricengine.ErrRecordsLayout` is exported, and the records-store refusals of `originNamespaceAt` match it through `sentinelError`.
  `fetchRunForCatchUp` returns such an error as is.
  A `TestHubCatchUp` row unsets the pair's records fetch refspec and asserts the refusal names the refspecs and is no `ErrOriginUnreachable`.
  The row sets `BOARD_SKIP_GIT` so the Bolt pull, which shares the store's config, does not fail first.
  It adds the pair with `SkipPush` so its records worktree's layout is not cached before the refspec is unset.
- `fabricengine: fix F3`: `TestOneRepositoryPairTips_RefuseARecordsStoreThatLostItsFetchRefspecs` covers `FetchPairTips` and `ReturnToPreviousTransition`.
  It asserts the error names the refspec restore and the tracking refs did not move.
- `fabricengine: fix F5, F6, F7, F10`: `RefuseHubWithoutBolt` names the join and the seed for a one-repository hub and carries the layout error.
  `cloneResetCommand` keeps `--force-bootstrap`.
  `TestCommand_HubWithoutItsBoltNamesTheResetClone` is a table with a two-repository row.
  The cleanup envelope test asserts `plain_archive_tags` is `[]` on a two-repository hub.
  A held-lock row asserts the refusal names a given rerun.
- Four self-check commits, below.
- `fabriccli: fix gate`: the gate refused two comment lines the round added, at 120 characters or more.
  `fabriccli/clone.go` now breaks its comment at the sentence boundary.
  `codetipsettle_integration_test.go` shortens its test doc to under 120 characters.
  A check of every comment line this round added found no other over-long line.

`_lym/landing/summary.md` carries F8 and F9, and the self-check's two summary fixes.
It sits under `_lym`, so it has no commit.

The gate's reach report then listed `hubCancelDeps`, `discardTaskBranchIfPresent`, `ReturnToPreviousTransition`, `fastForwardWeft` and `DiscardTaskBranch` as reached by no mapped test.
I added no test for it.
`ReturnToPreviousTransition` and `DiscardTaskBranch` already have tests that this round extended, so the map is missing them.
The `fastForwardWeft` and cancel rows stay out for the reasons given under F3 and F7.
`discardTaskBranchIfPresent` only passes an empty rerun, which a new test would pin as a literal.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: the summary's O4 bullet and Merge danger lacked `plain_archive_tags_error`; fixed.
- fix-consistency, LOW: `fabricengine/doc.go` O1 text was stale for the join and seed; fixed.
- fix-consistency, LOW: `fabriccli/doc.go` clone text was stale for the join and seed; fixed.
- fix-consistency, LOW: `fabricengine/doc.go` did not name `ErrRecordsLayout`; fixed.
- fix-affected-set, LOW: `fabriccli/clone.go:34` comment spelled `refs/lym/`; fixed.
- fix-affected-set: the `fastForwardWeft` read has no test; recorded under F3 as fixed differently.
- fix-proposal, LOW: any `originNamespaceAt` error read as a one-repository hub in `RefuseHubWithoutBolt`; fixed, only an `ErrRecordsLayout` match does, and every error is still carried.
- fix-proposal, LOW: `plain_archive_tags` is absent when the listing fails and nothing said so; fixed in the help and `doc.go`.

No fork returned without a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
