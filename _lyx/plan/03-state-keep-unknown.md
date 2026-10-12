# Card 3 — Keep the keys a record's type does not know on every state write

**Create:**
- `internal/state/keep.go`
- `plan:internal/state#keepUnknown` -> `func keepUnknown(t reflect.Type, oldRaw, newRaw []byte) []byte`
- `plan:internal/state_test#TestWriteJSON_KeepsUnknownKeys` -> `func TestWriteJSON_KeepsUnknownKeys(t *testing.T)`

**Edit:**
- `internal/state#writeJSONUnlocked`
- `internal/state/doc.go`
- `internal/shedengine#Shed.persist`
- `internal/shedengine#TestRun_UnknownTopLevelKeyBeforeRead`
- `internal/shedengine#TestRun_ExternalWriteSurvivesPersist`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/shedengine#TestRun_StrayExternalKeyDestroyed`

**Intent:**
D2 and D3: `WriteJSON` and `UpdateJSON` keep, on every write, each key of the file they replace that the record's Go type does not declare.

`writeJSONUnlocked`, which both call under the write lock they already hold, marshals `v` compact, reads the file at `path`, and when it exists and parses as JSON passes both through `keepUnknown(reflect.TypeFor[T](), old, new)` before indenting the result with two spaces, the indentation `json.MarshalIndent` gives today, and writing it atomically as now.
An absent file or one that does not parse adds nothing: the new value is written alone.
`UpdateJSON` keeps its abort on a decode failure before `mutate`, unchanged.

`keepUnknown`, in the new `keep.go`, merges `oldRaw` into `newRaw` by `t`:
- A type implementing `json.Marshaler` or `json.Unmarshaler` on its value or pointer, `json.RawMessage`, an interface, or any kind other than struct, pointer, map, slice and array is opaque: `newRaw` is returned whole.
- A pointer recurses into its element type; a `null` on either side returns `newRaw`.
- A struct whose two sides are both JSON objects keeps the new object's members in their order, recursing into each member whose key both sides hold with that field's type, then appends, in the old file's order and with its raw value, each old member whose key the type does not declare.
  Declared keys are the json tag names and field names, embedded structs' promoted fields included and `json:"-"` fields excluded, matched case-insensitively as `encoding/json` matches them, so a key known only by a case variant is decoded into its field and never kept.
  A key present in the new encoding is never overwritten.
- A map whose sides are both objects recurses into each value under a key both sides hold, with the map's element type; it adds no key the new map lacks.
- A slice or array whose sides are both arrays recurses into element i only when old element i, decoded into the element type and re-encoded compact, equals new element i compact; otherwise new element i is kept as it is, so an element updated in place, removed or moved loses its unknown keys and an untouched or appended neighbour keeps its own.
- A side of the wrong JSON shape returns `newRaw`.
Bound: only keys already in the file being replaced pass through, and each write costs one extra read of that file under the held lock.

`TestWriteJSON_KeepsUnknownKeys`, in `state_test.go`, is one table over a local fixture record, with no feature-package import.
The record holds an embedded struct whose fields are promoted, a slice of entry structs, a map of entry structs, a pointer to an entry struct, an `any` field, a field whose type implements `json.Marshaler` and `json.Unmarshaler`, and a `json.RawMessage` field.
Each row seeds the old file text, writes through `WriteJSON` or `UpdateJSON`, and compares the written file to an exact expected text.
Rows:
- an old file with an unknown top-level key and an unknown key in its one slice entry, written back by an `UpdateJSON` that appends an entry, keeps both keys with their exact values, the appended entry carrying none; a `WriteJSON` of the value read back with a top-level known field changed keeps both too;
- an `UpdateJSON` that changes a known field of entry 0 of two entries that each carry an unknown key drops entry 0's key and keeps entry 1's;
- removing entry 0 drops the unknown key of the entry that moved from index 1 to 0;
- a map value replaced under the same map key with different known fields keeps its unknown key, as D3's bound states, and a new map key gets none;
- an old file holding the key `Name` for a field tagged `name` writes `name` once, with the new value, and no `Name`;
- a key of a promoted field of the embedded struct is known: written once with the new value, never appended as unknown;
- extra keys inside the `any` field's object, inside the Marshaler field's object and inside the `json.RawMessage` object are not carried: each is written exactly as the new value encodes it;
- the pointer-to-struct field keeps its unknown key;
- a `WriteJSON` over a malformed file writes the new value alone.
A write over an absent file is already reached by `internal/state_test#TestWriteJSON_RoundTripOverwriteAndLayout`'s first write, and `UpdateJSON` over a malformed file is pinned by `internal/state_test#TestCorruptFile`.

The shed status run over unknown keys, the acceptance criterion's `status.json`, is pinned in `internal/shedengine`'s existing tests:
- `TestRun_UnknownTopLevelKeyBeforeRead`, which card 2 flipped to run, seeds one history entry, of producer `Earlier` so the run's own entry never folds into it, carrying an unknown key beside the top-level `unexpected_key`; after `Run` its raw map holds both keys with their values, the history entry the persist appended holds none, and `readStatus` reads the file.
  That covers the read gate, the persist's `UpdateJSON` and the history append in one run.
- `TestRun_ExternalWriteSurvivesPersist`'s `externalUpdate`, which runs inside producer A, also writes an unknown top-level key, and the test asserts the key is present after the persist.
- `TestRun_StrayExternalKeyDestroyed` pins the opposite of D2, so it is deleted.

`Shed.persist`'s doc paragraph "The accepted leniency asymmetry" says an external actor's unknown top-level key is destroyed by the full-struct marshal; it becomes: the persist keeps every key its type does not know, through `internal/state`.
`internal/state/doc.go` gains a section stating D1 to D3: no read refuses an unknown field, every write keeps the old file's unknown keys by the rules above, opaque types are written whole, and the bound.
`internal/state`'s budget row rises from 8 to 9; `internal/shedengine`'s stays, one test fewer.

**ImpactSummary:** every record written through `internal/state` now keeps unknown keys, across fabric, reed, orch, shed, loom, batten, webster, treadle and parentreview; `board.json` stays opaque through its Marshaler, as card 1 pins.

**Commit:** 3: keep unknown keys on every state write
