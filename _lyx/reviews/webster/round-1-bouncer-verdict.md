---
verdict: APPROVED
rationale: "Range 724b78b50..HEAD (parent main) implements card 1: every Remove refusal, both dirtiness checks and their status probes included, returns before archiveWeftTip with an empty record, force gates only those two checks, the four Delete targets have no remaining references, and the report's F1-F5 comment findings are fixed in comment-only commits 8cdbdee18..1599d4717 with go build, vet and the integration Remove/MutationRecord/CrossProduct tests passing."
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`, taken from `_lyx/shed/archive-tag-before-dirty-check/status.json`'s `product.parent: main`.
It holds the card-1 commit `015c710c2` and the five fixer commits for the round-1 report's findings.

## Report ratified

The round-1 report is APPROVED with four LOW and one NIT finding, and it has no `## Focus departures` section.
Its settled focus directives hold on re-check:

- `Topology.Remove` returns from slug, prime, target-exists, both merge checks, the warp probe and dirty check, and `refuseDirtyWeftWorktree` before `archiveWeftTip`, so each refusal returns an empty record.
- `force` gates exactly the two dirtiness blocks and never the archive.
- The dropped "worktree dir already gone" rationale was dead, since a missing target already refuses with "worktree not found".
- `TestRemove_ReusesSameTipArchiveTag` pins `archiveWeftTip`'s reuse path.
- `TestRemove_WarpStatusProbeFailurePushesNoTag` reaches the warp status probe.

## Fixes verified

- F1-F4: the rewritten headers and test docs in `remove.go`, `livestate_verbs_test.go`, `mutation_record_integration_test.go` and `remove_refusal_remedy_integration_test.go` now put one sentence per line.
  Their remaining breaks fall at clause boundaries: ", and only then", ", and an Add", ", so an operator".
  This satisfies CONSTRAINTS.md's Comment Line-Break Convention.
  The untouched fixed-column lines around them are legacy and out of scope.
- F5: `refusalof_test.go` now names "Remove's dirty pre-flight refusal", and no "anomaly path", "refusal-with-side-effects" or "tranche-1 refusal" prose survives outside `_lyx`.
- `git diff 015c710c2..HEAD` changes only `//` comment lines.

## Mechanical checks

- Delete group, assert-no-callers: `nameStrandedPortalTeardown`, `TestRemove_ResumedRemoveReusesArchiveTag`, `TestRemove_RefusalWithNothingStrandedOmitsRemedy`, `portalAndLauncherPermittedRoots` and the `archivedEntries` local have no references outside `_lyx`.
  The plan's `! grep -rn nameStrandedPortalTeardown` check passes.
- Edit group blast radius: `battencli`'s `teardownRefusal` matches only the `ErrArchiveFailed`/`ErrPairSiblingDirty` sentinels, and its doc remains accurate.
  No production caller depends on the dropped reconcile suffix.
- Checks run by this judge: `go build ./...` and `go vet -tags integration ./internal/fabricengine/`, plus `go test -count=1` for `fabricengine`, `battencli` and `fabriccli`, plus `go test -count=1 -tags integration ./internal/fabricengine/`.
  All of them pass, including every `TestRemove_*`, `TestMutationRecord_Remove*` and `TestCrossProduct` case.

Nothing blocking remains.
