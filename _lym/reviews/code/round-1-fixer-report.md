---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The bench pair's processes are ended with the teardown's own endOwned, not with EndSubjectProcesses, which serves a subject another machine owns, logs that on every call and spares a different set of roles. The end runs before the archive, not only before td.Run, so the archive holds what the processes committed."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The door is one-way as proposed. The blast radius is the one word fabric, since contracts/specs/loom-final-summary-spec.md allows one word and the description check refuses more; the guard and teardown effects are named in prose under it."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "No row for add slug-b1 slug-b2: the verb's cobra.MaximumNArgs(1) refuses two arguments before the guard runs. The reachable case of the len(args) check is no argument at all, which the NoSlug row pins."
  - id: F9
    disposition: fixed-differently
    reason: "A commit on task-b1 alone would make the first pair fail and leave the removed list empty, pinning nothing. The row instead puts the commit on task-b2 and refuses only task-b2 refs, so task-b1 is removed first and the list is checked against it."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

# Round 1 fixer report

All findings are fixed.
Tier 0 and tier 1 tests of the changed packages pass.
The new run-lock row was not run against the unfixed code.

## Changes

- F1: `RemoveBenchPairs` ends each bench pair's processes before archiving it; a `TestRemoveBenchPairs` row holds the bench run lock from a live registered process.
- F2: the bench removal moved from the `Remove` closure to the `Push` closure in `internal/battencli/wire.go`.
- F3: `benchPairAdmitted` bars a slug a batten run directory names; `hubgeom.PrimeWithoutGit` is exported to place the prime.
- F4: `_lym/landing/summary.md` updated, Merge danger included.
- F5: `TestWiredTeardown_RemovesTheBenchPairsBeforeTheTaskPair` drives the wired teardown row.
- F6: a half-removed bench pair is logged at Warn with `lym fabric prune --apply`.
- F7: `benchPairVerb` returns the reason, which the refusal carries in place of the way forward.
- F8: guard rows for an undecodable `run.json` and for no slug.
- F9: the refused-archive row checks the partial result.
- F10: the push admission shares `benchPairAdmitted`, so it also bars a slug a batten run names.
- F11: a bullet in `pattern/PATTERN-pair-remote.md`.

## Gate repair

The gate found wrapped and over-long comment lines that this round had added, and ran no test.
Each flagged comment is rewritten as shorter sentences, one per line, with no code change:

- `internal/battencli/teardownpush_integration_test.go`: the file header and the `TestWiredTeardown_RemovesTheBenchPairsBeforeTheTaskPair` comment.
- `internal/battenshed/doc.go`: the bench removal sentence.
- `internal/fabriccli/roleguard.go`: the `isBenchPairPath` comment.
- `internal/pairteardown/leftover.go`: the `RemoveBenchPairs` comment.
- `internal/pairteardown/leftover_integration_test.go`: the `holdRunLockFromLiveProcess` comment.

Tier 0 tests of those packages pass after the repair.

## Self-check

- fix-proposal, MEDIUM (bench pairs removed with no prime lock, against the `battenshed` doc sentence on registry writes): disputed, premise false. The sentence says which of Worktree-Create's steps its hold covers, not that every registry write takes the lock: the dobby's orphan removal (`internal/beamcli/dobbydeps.go`, `RemoveOrphanPair`) and `lym fabric remove` write the registry with no prime lock.
- fix-code-correctness, LOW (the same, noted outside its lens): disputed, premise false, as above.
- fix-code-correctness, LOW (false log line from `EndSubjectProcesses`): fixed.
- fix-consistency, LOW (a bench removal failure reads as a failed push): fixed; the reason now reads "the push step failed" and the `TeardownDeps.Push` comment covers it.
- fix-consistency, LOW (pattern bullet silent on the ordering exception): fixed.
- fix-consistency, LOW (false log line): fixed.
- fix-consistency, LOW (stale `roleguard.go` header): fixed.
- fix-consistency, LOW (stale `autonomy.go` header): fixed.
- fix-proposal, LOW (stale `TeardownDeps.Push` and `wire.go` comments): fixed.
- fix-proposal, LOW (false log line and missing pattern exception): fixed.
- fix-proposal, LOW (the guard refuses every bench verb where refs are not files): fixed; it lists the prime through git there. No test covers that branch.
- fix-affected-set, LOW (stale `roleguard.go` header): fixed.
- fix-affected-set, LOW (false log line and a differing spare list): fixed.
- fix-affected-set, LOW (archive tags pushed with pushes off, unstated): fixed in `internal/battenshed/doc.go` and the change description; the behaviour is unchanged.

## Disputed

No review finding is disputed.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this session.
