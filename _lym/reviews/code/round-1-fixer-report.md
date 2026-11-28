---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

# Fixer report, round 1

All paths are under `internal/landingshed/`.
The package passes its tests after the last commit.

## Changes

- **F1** (`e938587a9`): `parentMergeTip` asks `ParentAheadByOwed` again under the parent lock for a parent first read as owed-ahead.
  On false it returns the upstream tip read before the lock.
  Both reads share one error wording.
- **F2** (`e938587a9`): added `TestFinalize_LockFreePassMergesTheUpstreamTipOfAnAheadParent`, with rows for owed false, a nil seam and an owed answer that turns false under the lock.
  The last row failed before the F1 change.
  The rows sit in their own table, since the locked pass ends them in a non-halting Stuck that neither named table expresses.
- **F5** (`e938587a9`): the `parentMergeTip` doc no longer says "never the local tip" and describes the second read.
- **F3** (`d3bad1e1e`): added an owed-ahead row to `TestMergeIn_MergeTip`.
  It failed with `true` passed at the Parent-Align call.
- **F4** (`cae2038c9`): `gateUnlocked` takes no owed local tip for a landing known to need GitHub; the docs in `finalize.go` and `doc.go` name the exception.
- **Self-check follow-up** (`f0c98f75b`):
  `needsGitHubOnDisk` returns a failed read as an error, and `gateUnlocked` then merges the upstream tip.
  Added rows to `TestFinalize_LockFreePassStops` for the F4 behaviour, the failed need read and the failed second owed read.
  The first two failed with `true` passed at the `gateUnlocked` call.
  Corrected the `doc.go` sentence and stated the repository-rules limit in the `gateUnlocked` doc.
- **Comment-wrap repair** (`5e68647d8`): rewrote two comments that broke mid-sentence, one in the `gateUnlocked` doc and one on the `owedParent` helper in `finalize_unlocked_test.go`, as one sentence per line.
  No code changed.

## Self-check

- fix-code-correctness, LOW, a pull request only repository rules require still merges owed tips: fixed differently (limit stated in the doc; closing it needs a GitHub read before the lock).
- fix-code-correctness, LOW, a failed on-disk read still merges owed tips: fixed.
- fix-consistency, LOW, `doc.go` "always" overclaims: fixed as proposed.
- fix-proposal, LOW, the repository-rules case: fixed as proposed (stated limit), same change as the first line.
- fix-affected-set, MEDIUM, the repository-rules case: fixed as proposed (residue stated here and in the doc), same change as the first line.
- fix-affected-set, MEDIUM, no test fails if the F4 argument reverts: fixed as proposed.
- fix-affected-set, MEDIUM, `doc.go` false under the push skip or no upstream: fixed as proposed, same change as the consistency line.
- fix-affected-set, LOW, the second owed read's error has no test: fixed.

Remaining limit: a landing that needs GitHub only because of the repository's rules, on an owed parent, still merges and gates the owed tips before `parentPushBlocks` ends the call.

## Disputed

None.

## Deferred

None.

## Rulings

None.
