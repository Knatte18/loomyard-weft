# Leave the caller's payload map unchanged on board upserts and merges

`TestDiscussedPayload` in `internal/boardengine` could crash with `fatal error: concurrent map writes`.
The store's payload checks drop a key that equals the stored value,
and they dropped it from the caller's own map.
The test shares one payload map across its parallel upsert, batch and merge subtests, so those deletes raced.

`Store.UpsertTask`, `Store.UpsertTasksBatch` and `Store.MergeTasks` now clone each payload before checking it.
A caller's map is never changed by these calls, and callers may share one payload across concurrent writes.
`TestDiscussedPayload` now asserts that every path leaves the shared payload unchanged.

Closes #289

## Design decisions

- The fix is in the store rather than in the test, so the guarantee holds for every caller of the three methods.
- The checks still delete no-op keys, now from the store's own copy.
- The copy is a shallow `maps.Clone`: it protects the map's keys, not the values nested in it.
- `MergeTasks` already cloned the upsert map before writing the carried keys into it; that clone now comes first,
  ahead of the checks.

## Merge danger

**Door:** two-way
**Blast radius:** board
