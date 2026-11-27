# Card 6 — layout-mutation-record

**Create:**
- `plan:internal/fabriclayout#Mutations` -> `type Mutations struct`
- `plan:internal/fabriclayout#Mutations.Entries` -> `func (m Mutations) Entries() []Mutation`
- `plan:internal/fabriclayout#Mutations.Len` -> `func (m Mutations) Len() int`
- `plan:internal/fabriclayout#Mutations.MarshalJSON` -> `func (m Mutations) MarshalJSON() ([]byte, error)`
- `plan:internal/fabriclayout#MutationsOf` -> `func MutationsOf(entries []Mutation) Mutations`
- `plan:internal/fabriclayout#MutationRecord` -> `type MutationRecord struct`
- `plan:internal/fabriclayout#MutationRecord.Mutated` -> `func (r MutationRecord) Mutated() Mutations`
- `plan:internal/fabriclayout#TestMutations_MarshalJSON` -> `func TestMutations_MarshalJSON(t *testing.T)`
- `plan:internal/fabriclayout#TestMutationRecord_EmbedsAndMarshalsUnderMutationsKey` -> `func TestMutationRecord_EmbedsAndMarshalsUnderMutationsKey(t *testing.T)`
- `plan:internal/fabriclayout#TestMutationsOf` -> `func TestMutationsOf(t *testing.T)`
- `plan:cmd/lym#isMutationRecordEmbed` -> `func isMutationRecordEmbed(firstField string) bool`
- `plan:cmd/lym#TestIsMutationRecordEmbed` -> `func TestIsMutationRecordEmbed(t *testing.T)`

**Edit:**
- `internal/fabricengine#`
- `internal/fabriccli#`
- `internal/stencilcli#`
- `internal/pairteardown#`
- `internal/hibernatecli#`
- `internal/hubforge#`
- `cmd/lym#TestMutationRecord_FabricengineProductionSource`

**Intent:** Split the mutation record into the plain record data `internal/fabriclayout` holds and the recorder `internal/fabricengine` keeps (D1: the layout holds `MutationRecord` as plain data that serializes as today; the engine keeps the recorder that appends and logs entries).

In `internal/fabriclayout/mutation.go`:
- `Mutations` is the record value: an unexported `entries []Mutation` and the value-receiver `Entries`, `Len` and `MarshalJSON` moved from today's recorder, unchanged.
  `MarshalJSON` still emits `[]` for an empty or zero record.
- `MutationsOf` returns a record holding a copy of entries, the one way the engine hands its entries over.
- `MutationRecord` is today's carrier, its one field `Mutations Mutations` with the `json:"mutations"` tag, and `Mutated` returns it.

In `internal/fabricengine` (the whole package, so every caller of the changed methods is covered):
- `Mutations` stays the recorder: `hubRoot` and its entries, `NewMutations`, `Append` and `AppendRef`, which log every entry, and `Extend`, `Entries` and `Len`.
  `Snapshot` returns a `fabriclayout.Mutations` built through `MutationsOf`, and `Extend` takes a `fabriclayout.Mutations`, appending its `Entries()`.
  The recorder's `MarshalJSON` goes, since only the record value is marshalled.
- Every result type embeds `fabriclayout.MutationRecord` as its first field, and the engine's `MutationRecord` declaration goes.
- `doc.go`'s mutation record section says the entry, the kinds, the record value and `MutationRecord` live in `internal/fabriclayout` and the recorder here; its rules are unchanged.
- In `mutation_test.go`, `TestMutations_MarshalJSON` and `TestMutationRecord_EmbedsAndMarshalsUnderMutationsKey` move to the layout as the tier0 tests above, with their rows and literal wants.
  They build their records through `MutationsOf` over literal `Mutation` entries, or a zero `Mutations{}`, instead of the engine's recorder, which the layout cannot import.
- `TestMutationsOf` (tier0) changes the input slice after the call and wants `Entries()` unchanged, and changes the slice `Entries()` returned and wants the next `Entries()` unchanged, so each copy the record value promises fails the test if it is dropped.
- The recorder's tests stay and read through `Snapshot().Entries()` where they read a record value.
  No test is deleted here: each guards one behavior of either the record or the recorder.

Outside fabric, every spelling of the record value or the carrier, `fabricengine.Mutations` as a value type and `fabricengine.MutationRecord`, becomes the `fabriclayout` type:
the envelope helpers of `internal/fabriccli` and `internal/stencilcli`, `internal/pairteardown`'s teardown record and tests, `internal/hibernatecli`'s reconcile test and `internal/hubforge`'s `Hub` field.
A `*fabricengine.Mutations` recorder parameter stays as it is.
Each listed package's `doc.go` is covered by its package glyph and changes only where it names the moved type.

A recorder literal such as `&fabricengine.Mutations{}` or `new(fabricengine.Mutations)`, in `internal/hubgeom`'s and `internal/gatecli`'s tests among others, names the recorder, which keeps its name, and stays as it is.

`cmd/lym#TestMutationRecord_FabricengineProductionSource` accepts `MutationRecord` or `fabriclayout.MutationRecord` as a mutating result type's first field, and refuses both for a read-only one.
It decides through `isMutationRecordEmbed`, a `_test.go` helper that reports whether a first-field spelling is one of the two.
`TestIsMutationRecordEmbed` (tier0, table) wants true for `MutationRecord` and `fabriclayout.MutationRecord`, and false for `fabricengine.MutationRecord`, `Mutations` and an empty spelling, so a wrongly widened or narrowed comparison fails it.
Its `//lym:watch` adds `internal/fabriclayout/*.go`, since cards 10 and 11 move result types there.

No behavior changes (D5): every verb's JSON carries the same `mutations` array, and the recorder logs each entry as before.

**ImpactSummary:** The recorder keeps its name and call shape, so its call sites compile unchanged; the record value's few spellings outside fabric, in five packages, and the result-type guard change.

**Commit:** `6: layout-mutation-record`
