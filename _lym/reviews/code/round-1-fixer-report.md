# Fixer-report, round 1

## Changes

All five findings fixed as proposed; none disputed or deferred.

- **F1** (`internal/battencli/owner_test.go`): added row `TeardownWithPairPresentRunsTheReconcile` with a `pairPresent` flag that builds a `.git` holding `HEAD`, `objects` and `refs` at `fabricengine.WorktreePath`, which `lymcwd.ResolveWorktree` accepts. The row fails if the `!present` condition is dropped.
- **F2** (`pattern/PATTERN-owner-machine.md`): added the exception to "Origin reconcile first" and to the passes bullet. `PATTERN.md`'s entry still holds and is unchanged.
- **F3** (`internal/battenshed/doc.go`): added the sentence after the unreachable-origin sentence.
- **F4** (`owner_test.go`): added row `UnownedResumes`.
- **F5** (`owner_test.go`): extended the file header.
- **Self-check follow-ups**: qualified the origin-owed marker bullet in `PATTERN-owner-machine.md`, and renamed the test to `TestOwnerPreflight_TeardownResume` since it now holds a pair-present row.

`lym gate test ./internal/battencli` is ok.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (marker bullet at line 29 unqualified): fixed as proposed.
- fix-consistency, LOW (header and line 13 do not name the new case): not changed; they state the gating verdict and the unreachable-origin case, and make no claim that every resume reconciles. Disposition: premise false for a claim, the exception is stated in "Origin reconcile first".
- fix-proposal, MEDIUM (marker bullet contradicts the exception): same as the consistency finding, fixed.
- fix-proposal, LOW (test name): fixed by renaming.
- fix-proposal, LOW (`pattern/PATTERN.md` unchecked): the index is the root `PATTERN.md`, line 27; its text still holds, unchanged.
- fix-affected-set: no findings; its marker-bullet note is the same finding, fixed.

## Disputed

None.

## Deferred

None.
