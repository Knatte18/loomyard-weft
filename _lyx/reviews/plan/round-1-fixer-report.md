# Fixer report — plan, round 1

## Changes

- F1 (MEDIUM): `00-overview.md`'s third Shared Decisions bullet now opens by marking itself a departure from D2, pending an `Added after Discussion` entry under D2 in the decision record, instead of presenting the save-a-copy wording as the plan's own decision.
  Card 3's form 1 wording stays as it is.
  The decision-record entry itself is deferred; see below.
- F2 (LOW): card 3's `TestFinalize_OverRealHub` fourth step now expects the parent's code checkout path as `filepath.EvalSymlinks(parentCode)` returns it, since the guard error's `Checkout` is the symlink-resolved path.
- F3 (LOW): card 3 gains a sentence after form 2's reaching paragraph.
  It says form 3 is reached when the guard's unfetched read of the local upstream ref shows a divergence at step 4 that step 1b3's origin reconcile did not stop, because it arose between 1b3 and step 4 or because no reconcile is wired.
  It names the not-synced row of `TestFinalize_StuckReasons` as the reaching test.
- F4 (NIT): card 5's `parentCodeDirtFromFabric` bullet drops "with no git".
  It now says the `recordedParentBranch` row spawns nothing, and the `OpenParent` row reaches its error through a failing `git worktree list`, a git spawn that `loomcli`'s `TestMain` already makes hermetic through `gitkit.HermeticGitEnv()` per `PATTERN-test-isolation`.
  Fixed differently in one part: the proposal said the row runs "under `gitkit.HermeticGitEnv()`".
  That reads as a per-row call, but `internal/gitkit/hermetic.go` says to call it once as the first line of `TestMain`, and `internal/loomcli/testmain_test.go` already does.
- F5 (NIT): card 4's Edit list gains `internal/haltnote#parentHeader` after `internal/haltnote#Parse`.
- Self-check: card 2's `TestStatus_MergesUncommittedChangesBothSides_ExcludesWeftArtifacts` bullet now compares the returned checkout after `filepath.EvalSymlinks` on both sides. This applies F2's fix to the same comparison in card 2.

`lyx loom validate-plan` reports `ok` after every change.

## Self-check

- fix-consistency, LOW: card 5's `HermeticGitEnv` clause reads as a per-row call. Fixed with my own change, merged with the fix-proposal finding: it now names `loomcli`'s `TestMain` as the place that makes the spawn hermetic.
- fix-claims: no findings.
- fix-proposal, LOW: the same `HermeticGitEnv` wording, a duplicate of the fix-consistency finding. Fixed as proposed.
- fix-affected-set, LOW: card 2 is F2's untouched twin, comparing `CodeDirtyPaths()`'s checkout without symlink resolution. Fixed as proposed.

## Disputed

None.

## Deferred

- F1, the `Added after Discussion` entry under D2 in `_lyx/discussion/decision-record.md`.
  The review's fix has me send the correction to `ly:orch`, but my prompt forbids sending any message to another session about the review.
  The decision record is also outside my write surface, which is `_lyx/plan` and this report only.
  The loop owner or `ly:orch` needs to add the entry.
  It should record that D2's pre-abort step now reads "save a copy of any change of their own outside the checkout".
  The reason: a plain git commit there moves HEAD off the record's start, so `sideConcludeMayHaveLanded` (`internal/fabricengine/mergeguards.go`) makes `--abort` refuse with `merge conclude already landed`, and a `git stash` takes the staged merge with it.
