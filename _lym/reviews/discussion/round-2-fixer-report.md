# Round 2 fixer report — discussion

Every finding's premise held against the record and the code; all ten are fixed in `_lym/discussion/decision-record.md`, with the round logged in `_lym/discussion/support-log.md`.

## Changes

- F1 (BLOCKING): fixed as proposed.
  D5 gives a package whose coverage build fails, or which has no test file or no `tier0` or `tier1` test, a map with its Go file hashes and no tests, with the failure as its cause, counting against `--max`.
  It is stale again only on a file change or past the age limit.
  D4 lists the failure field.
  New D5 criterion added.
  The self-check extended it: see Self-check.
- F2: fixed as proposed.
  D6's stop sentence now says the prime dobby's stop also calls `runproc.EndRole` for `runproc.HubOwner` with `runproc.RoleTestMap`.
  The D6 criterion names that route.
- F3: fixed, with the self-check narrowing one part.
  D7 step 6 records the function hash of every test step 1 returns, judged or not.
  D8 matches HEAD's step 1 set against that recorded set.
  The D7 repeat-call and D8 criteria now cover a diff that also changes a `tier2` test.
  The proposal's Go file hashes for every package holding a step 1 test were made, then reverted to judged packages only: see Self-check.
- F4: fixed as proposed, naming a new key.
  D5: each test runs in its own process group, and a timed-out test is ended with its whole group, with a bounded grace on its wait.
  Each coverage build and the `lym` build run under `redundancy_build_timeout_min` (new gate key, default 10), and a build past it counts as failed.
  D5's time bound now leaves the slot wait unbounded by the verb.
  D6's orphan sentences are narrowed to once the child holds its slot.
  D7 step 4 cites the same build timeout and process-group end.
- F5: fixed as proposed.
  D7 step 1 looks each current test up by package and name across the package's `_test.go` files at the merge base, and passes over a deleted `_test.go` path.
  The rename case was added to the D7 merge-base criterion.
- F6: fixed as proposed.
  D3's Bound now says an inserted duplicate block shifts every later identical block's ordinal, and that review bounds the resulting wrong covers.
- F7: fixed as proposed.
  D5's Bound names a change to a dependency or to a non-Go file of the package directory.
- F8: fixed as proposed.
  D7's Bound names any change outside the map test's function text, from the diff or landed since the map was built.
  Step 6 names the map commit and age for every judged package, and the D7 criterion matches.
- F9: fixed as proposed.
  D1 states the spawn scan moves from `cmd/testtiming/spawnscan.go` into `internal/testcover`, and D7 step 2 cites it there.
- F10: fixed as proposed.
  The criterion reads "no map test and no other judged test covers".

## Self-check

- fix-consistency, LOW, D4 holds no field for D5's build-failure cause: fixed as proposed, adding a D4 bullet.
- fix-consistency, LOW, D7's repeat-call cost still says "the changed test files" after step 1 reads each package's test files: fixed as proposed.
- fix-consistency, LOW, `redundancy_build_timeout_min` sits only under Q7 though it also bounds the step: fixed as proposed.
  Q6 now holds the key and its default, and Q7 points at it.
- fix-claims, LOW, step 1 claims renamed or moved files' tests are not taken, but a move across packages is taken: fixed as proposed.
  Step 1 now says "renamed or moved within its package".
- fix-claims LOW and fix-way-forward LOW, merged: no route lists a package's `_test.go` files at the merge base, and an absent directory has no outcome.
  Fixed as proposed.
  Step 1 lists them through `fabricengine.CodeDirAt` (new), and a directory absent at the merge base holds no tests.
- fix-way-forward MEDIUM and fix-proposal MEDIUM, merged: a failed-build empty map is never retried while files are unchanged, is never reported, and D7 judges against it silently.
  Fixed with my own change.
  I did not take the proposal's retry-within-the-next-batch wording, since a retry on every batch at one HEAD brings back F1's loop for a failure that persists.
  Instead:
  - a failed-build map is also stale once HEAD differs from its recorded commit, so it is retried once per prime HEAD move;
  - D5's exit lists it with the failure;
  - D7 reports such a package as not judged with the failure;
  - D7's repeat skip treats it as it treats a missing map.
  The D5 and D7 criteria were updated to match.
- fix-proposal, LOW, file hashes for every package holding a dropped `tier2` test make the report go stale on unrelated edits: fixed as proposed.
  Step 6, D8 and its criterion are back to file hashes per judged package.
  Function hashes still cover every step 1 test, which is all F3's premise needs.
- fix-proposal and fix-way-forward outside-lens notes: a hub-owner `runproc.EndSparing`, at an anchor or at a non-teardown session end, also ends a `test-map` child.
  This predates this round's change and the fix did not reach it, so it is out of the fix-check's scope.
  No change.

Every fork returned a report.

## Gate repair

The citation gate found four citations of things this task creates that were missing their ` (new)` mark: `internal/testcover` in D1 and D7 step 2, and `runproc.RoleTestMap` in D6 and its criterion.
Each now carries ` (new)`.
D7 step 2 was reworded to "D1's static spawn scan in `internal/testcover` (new)", so that the mark follows the path directly.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
