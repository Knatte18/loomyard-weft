# Drift: re-judge an uncaused loosened ruling once the range changes

`Drift-Verify` reused a stored file ruling while that file's own diff and the range's `Loosened-Check:` trailers were unchanged.
When a deleted test file was ruled `loosened` with `uncaused` checks,
a later commit that added those checks to another test did not change that key,
so the row halted again on the same ruling and spawned no judge.

Two things change:

- **Re-judging.** A ruling that holds an `uncaused` check, or names a carrying file,
  is now reused only while the range's full diff is also unchanged, so a later commit that answers it reaches the judge.
- **Moved checks.** A removed assertion that a test in another file still holds, on the same behaviour and no looser,
  is no longer a loosened check.
  The judge searches the repository's tests for it and lists the carrying file under the new `carried_by` field.
  The `Loosened-Check` instructions and the `Code-Review` rubric state the same rule.

The `Drift-Verify` halt now names a third way forward beside a trailer and a revert: commit the dropped checks in a test.

Closes #249

## Design decisions

- The new range key is a hash of the whole committed diff from the review base to `HEAD`, read through a new `RangeDiff` seam.
  The list of changed files holds only modified and deleted files,
  so a key built from it would miss checks moved into a new test file.
- The key is deliberately coarse: any change to the range's diff re-judges every file whose ruling depends on the range,
  including a change that has nothing to do with the dropped checks.
  The cost is an extra judge run.
- Rulings with no `uncaused` check and no `carried_by` are reused as before.
- `judged.json` gains `range_key` and `carried_by`.
  A stored `uncaused` ruling without a `range_key` does not match, so it is judged again at the next pass that lists files.
- `carried_by` is the judge's attestation: lym does not verify that the named file holds the check.

## To check by hand

- No live judge was run against the new `drift-template-judge` wording.
  The tests script the judge's answers, so they cover when a file is judged again,
  not whether a real judge finds a moved check or rejects a looser copy of one.

## Merge danger

**Door:** two-way
**Blast radius:** drift
