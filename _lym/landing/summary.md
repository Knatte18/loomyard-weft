# Board upserts and merges leave the caller's payload map unchanged

`TestDiscussedPayload` in `internal/boardengine` crashed now and then with `fatal error: concurrent map writes`.
The payload checks in `Store.UpsertTask`, `Store.UpsertTasksBatch` and `Store.MergeTasks` drop a key whose value equals the stored one,
and they dropped it from the caller's own map.
The test hands one row's payload map to its parallel upsert, batch and merge subtests,
so the subtests that passed the check deleted `discussed` from the same map at once.

## The fix

Each of the three store methods now clones its payload before checking it,
so a caller's map is never changed and callers may share one payload across concurrent writes.
`Store.MergeTasks` already cloned its payload before adding `issues`, `created` and `project`;
that clone moves to the top of the method, ahead of the checks.

`TestDiscussedPayload` keeps the shared payload and asserts that every path leaves it unchanged.
On the old code the `StoredValuePasses` row fails that assertion on each path every run, race detector or not.

## Design decisions

- The fix is in the store rather than in the test.
  A board method that deletes keys from the map its caller passed is the defect,
  and copying the payload in the test would have hidden it from every other caller.

## Merge danger

**Door:** two-way
**Blast radius:** board
