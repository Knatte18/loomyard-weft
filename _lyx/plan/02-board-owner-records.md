# Card 2 — board-owner-records

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#normalizeTask`
- `internal/boardengine_test#TestNewTask`
- `internal/boardengine#storeFile`
- `internal/boardengine#loadedStoreFile`
- `internal/boardengine#loadedStoreFile.UnmarshalJSON`
- `internal/boardengine#Store`
- `internal/boardengine#Store.Load`
- `internal/boardengine#Store.Save`
- `internal/boardengine#scopeChanged`
- `internal/boardengine#replayFields`
- `internal/boardengine#replayRun`
- `internal/boardengine#FabricReplay.Apply`
- `internal/boardengine#TestFabricReplay`

**Create:**
- `plan:internal/boardengine#Owners` -> `type Owners struct`
- `plan:internal/boardengine#ParseOwners` -> `func ParseOwners(boardJSON []byte) (Owners, error)`
- `plan:internal/boardengine#Owners.Effective` -> `func (o Owners) Effective(slug string, parentOf func(slug string) (string, error)) (string, error)`
- `plan:internal/boardengine#OwnedElsewhere` -> `func OwnedElsewhere(owner, here string) bool`
- `plan:internal/boardengine#TestOwnersEffective` -> `func TestOwnersEffective(t *testing.T)`

**Intent:**
D2's records on Bolt.
`Task` gains `OwnerMachine string` with JSON key `owner_machine,omitempty`; a note never carries one (`normalizeTask` clears it on a note).
`board.json` gains one top-level hub owner record, `prime_owner` (`omitempty`), declared on `storeFile` and `loadedStoreFile` so it is a known key, held on `Store` as `primeOwner` and round-tripped by `Load` and `Save`; an older binary keeps both keys through the existing `Unknown` maps, and a record an older binary wrote reads as unowned (`PATTERN-tolerant-records`).
`scopeChanged` keeps leaving `OwnerMachine` out of its comparison, and its doc comment names it beside `RunSettings` as a field a run-held entry takes, so the run lock never refuses an owner change.
`replayFields` gains `owner_machine` as a replay field, so a local owner write both sides changed keeps origin's value and is reported in `Lost`.
`FabricReplay.Apply` also replays `prime_owner`: it decodes the top-level `prime_owner` of the base, parent and commit bytes, and when the commit changed it, applies it unless origin's store changed it since the base to another value, in which case origin's value stands and a `Lost` finding with `Entry` `prime`, `Field` `prime_owner` and the local value is reported; an applied change sets `replayRun.applied`.
`ParseOwners` decodes `board.json` bytes into `Owners`: the prime owner, and per task its recorded owner, whether it is a task, done, and run-held (`IsRunStatus`); an empty input is a board with no entry.
`Owners.Effective` answers D2's effective owner: `""` (owned by no other machine) when no prime owner is recorded, for a slug the board does not hold, and for a run-held task with no owner recorded; the recorded owner when one is set; otherwise the parent's owner, `parentOf(slug)` naming the parent pair's slug or `""` for the prime, the prime's owner for the prime and the parent task's own effective owner for a pair parent; a `parentOf` error is returned.
`OwnedElsewhere` reports `owner != "" && owner != here`, the one comparison every owner check uses.
`TestOwnersEffective` is a table over `ParseOwners` bytes: empty bytes (no prime owner, no entry), undecodable bytes returning the error, no prime owner, recorded owner, run-held unowned, unowned task following the prime, unowned task following a pair parent's owner, an absent slug, a `parentOf` error, and `OwnedElsewhere` rows for empty, same and other owners.
`TestNewTask` gains a row: a note payload carrying `owner_machine` stores none.
`TestFabricReplay` gains rows: both sides changed `owner_machine` keeps origin's and reports the local one lost; only the commit changed it applies it, on a run-held entry, so `checkRunLocks` passes the owner change; both sides changed `prime_owner` keeps origin's and reports it lost; only the commit changed `prime_owner` applies it.

**ImpactSummary:** boardengine's record shape and replay only; callers of `Task` gain an omitempty field, no other package changes.

**Commit:** 2: board: owner_machine per task and the hub's prime owner on Bolt
