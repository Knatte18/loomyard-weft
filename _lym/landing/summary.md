# Drift-Verify re-judges an uncaused loosened ruling after a new commit

Drift-Verify stores the judge's ruling on each file of the review range in `judged.json`, under a key built from that file's diff and the range's `Loosened-Check:` trailers.
A later pass reused the ruling while that key held.
When a deleted test file's checks were ruled `uncaused`, a later commit that added those checks to another test left the deleted file's diff and the trailers unchanged.
The pass reused the stale ruling and halted again on the same checks, with no judge spawned.

## What changes

- `JudgedFile` gains an optional `range_key`: the sha256 of every file's path and diff in the range, in path order, plus the trailer lines.
  Every new file ruling stores it.
- `listFiles` reuses a stored ruling that holds an uncaused check only while the range's key is also unchanged.
  Any new commit in the range therefore has the judge rule on it again.
  Rulings with no uncaused check are still reused by the file's own key, so a commit elsewhere does not re-judge them.
- The halt over uncaused checks names a third way forward: commit the dropped checks in a test, then `lym loom resume`, which has the judge rule on the range's files again.
- The `driftengine` package doc states the new reuse rule and the way forward.

## Design decisions

- The re-judge is tied to any change in the range, not to the test files a ruling compares against.
  The ruling does not record which files it compared, and an uncaused ruling halts the run anyway.
  Under that halt, any new commit is the writer's attempt to answer it, so one judge spawn per new commit is the right cost.
- An uncaused ruling stored before this change has no `range_key`, so the first pass after the upgrade judges it once more.
  The field is optional, so older records still read.

## Tests

`TestDriftVerify` gains the case "a later commit that adds the dropped checks to another test has the uncaused ruling judged again".
It halts on an uncaused ruling and checks that the halt names the new way forward.
It then adds a second file to the range and checks that a second judge is spawned over both files and that the pass is Done.
On the merge base the second pass reuses the stored ruling and halts, so the case fails there.

## Merge danger

**Door:** two-way
**Blast radius:** drift
