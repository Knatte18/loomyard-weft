# Card 1 — board-unknown-keys

**Create:**
- `plan:internal/boardengine#jsonKeys` -> `func jsonKeys(t reflect.Type) []string`
- `plan:internal/boardengine#splitUnknown` -> `func splitUnknown(data []byte, known []string) (map[string]json.RawMessage, error)`
- `plan:internal/boardengine#appendUnknown` -> `func appendUnknown(object []byte, unknown map[string]json.RawMessage) ([]byte, error)`
- `plan:internal/boardengine#Task.MarshalJSON` -> `func (t Task) MarshalJSON() ([]byte, error)`
- `plan:internal/boardengine#Task.UnmarshalJSON` -> `func (t *Task) UnmarshalJSON(data []byte) error`
- `plan:internal/boardengine#storedEntry.UnmarshalJSON` -> `func (e *storedEntry) UnmarshalJSON(data []byte) error`
- `plan:internal/boardengine#storeFile.MarshalJSON` -> `func (f storeFile) MarshalJSON() ([]byte, error)`
- `plan:internal/boardengine#loadedStoreFile.UnmarshalJSON` -> `func (f *loadedStoreFile) UnmarshalJSON(data []byte) error`
- `plan:internal/boardengine_test#keptValue` -> `func keptValue(t *testing.T, raw []byte, slug, key string) (string, bool)`
- `plan:internal/boardengine_test#addEntryKeys` -> `func addEntryKeys(t *testing.T, boardPath, slug, members string)`

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#storedEntry`
- `internal/boardengine#storeFile`
- `internal/boardengine#loadedStoreFile`
- `internal/boardengine#Store`
- `internal/boardengine#Store.Load`
- `internal/boardengine#Store.Save`
- `internal/boardengine#ApplyPatch`
- `internal/boardengine#cloneTasks`
- `internal/boardengine#checkRunLocks`
- `internal/boardengine#scopeChanged`
- `internal/boardengine#TaskWithLayer`
- `internal/boardengine/doc.go`
- `internal/boardengine_test#TestLoad`
- `internal/boardengine_test#TestRunLock`

**Intent:**
A board.json rewrite keeps every entry key and top-level key this binary does not know (D1, D2), so an older long-lived run writing status no longer drops a newer binary's field.

Entry keys (D1), in `task.go`:
- `Task` gains `Unknown map[string]json.RawMessage` tagged `json:"-"`: the decoded entry's keys this binary does not know, each with its raw JSON value, nil when there are none.
- `jsonKeys` returns the JSON key names of a struct type's exported fields (the tag name, else the field name, skipping `-`), so the known set is derived from the struct and never a hand list.
- `splitUnknown` decodes a JSON object into `map[string]json.RawMessage` and drops every key that `strings.EqualFold`s a known name, the same case-insensitive match `encoding/json` uses to fill a field; it returns nil when nothing is left.
  So `"Title"` decodes into `Title` and is never also kept.
- `appendUnknown` writes the unknown members into an encoded object before its closing brace, in sorted key order, and returns the object unchanged when there are none, so an entry without unknown keys encodes byte-identically to today.
- `Task.UnmarshalJSON` decodes into a local plain type converted from `t`, which keeps the caller's pre-set defaults as `NewTask` relies on, then sets `t.Unknown` from `splitUnknown` with `Task`'s `jsonKeys`.
  `Task.MarshalJSON` encodes the plain type and `appendUnknown`s `t.Unknown`.
  No step decodes a kept value into a Go type, so a value such as 9007199254740993 keeps its exact digits; `MarshalIndent` may re-indent it, never change it.
- `ApplyPatch` unmarshals the encoded existing task into `map[string]json.RawMessage` instead of `map[string]any`, and overlays each patch field as its own `json.Marshal` encoding; the merged object decodes through `Task.UnmarshalJSON`, so the kept keys come back raw.
  Today's `map[string]any` step turns every number into a float64, which would round 9007199254740993.
  `ApplyPatch`'s doc comment names the raw-member overlay in place of "existing → map".
  `NewTask` is unchanged: its callers refuse a key outside `UpsertPayloadKeys` before it runs (D4).

Retired keys (D3), in `migrate.go`:
`storedEntry` embeds `Task`, so `Task.UnmarshalJSON` would be promoted onto it and `Tier` and `Type` would never decode.
`storedEntry.UnmarshalJSON` calls `e.Task.UnmarshalJSON`, decodes `tier` and `type` through a local struct holding only those two fields, then deletes from `e.Task.Unknown` every key that `EqualFold`s `tier` or `type`, setting it nil when empty.
The retired keys are consumed and never kept, so the saved entry carries neither, as today.
`storedEntry`'s comment says it consumes them.
Only `Load` decodes a `storedEntry`, and nothing encodes one.
`TaskWithLayer`, in `layer.go`, also embeds `Task`, so `Task.MarshalJSON` is promoted onto it and an encoding would drop `Layer`; nothing encodes one today, and its comment says it is never JSON-encoded for that reason.

Top-level keys (D2), in `store.go`:
- `storeFile` and `loadedStoreFile` each gain `Unknown map[string]json.RawMessage` tagged `json:"-"`.
  `loadedStoreFile.UnmarshalJSON` decodes through a local plain type, then `splitUnknown`s against `jsonKeys` of that type.
  `storeFile.MarshalJSON` encodes through a local plain type and `appendUnknown`s.
- `Store` gains an `unknown` field; `Store.Load` resets it and sets it from `file.Unknown` on the board.json path; `Store.Save` passes it as `storeFile.Unknown`.

`cloneTasks` also copies `Unknown` with `maps.Clone`, keeping its doc's promise that the copy aliases no map.
`checkRunLocks` and `scopeChanged` keep their bodies: a kept key is never compared, so a status write on a run-held entry carrying one passes (D4).
Their doc comments, which say "any field but Status" differs, gain beside the RunSettings sentence that `Unknown` is never compared, since this binary cannot change a kept key.
Every write path is load, mutate, save over `Task` values, so set-status, upsert, batch upsert, promote, prune and set-deps keep an entry's keys; `remove` drops them with the entry; a merge's upserted entry keeps its own through `ApplyPatch`, and each removed entry's go with it.
The store `version` check stays as it is (D6).
`validateUpsertFields` is unchanged, so an upsert payload carrying an unknown key is still refused with the existing message; `TestUpsertFieldAllowlist`'s `upsert_typo_key_errors` already pins it.

`doc.go` gains a `# Unknown keys` section before `# Migration`:
an entry's and the store's unknown keys are kept as raw JSON and written back after the known fields in key order on every write, while known fields beside them change;
the guarantee starts with this binary, and a process started from an older one still drops them (D5);
retiring a board field means naming it in the decode shape so it is consumed and dropped, as `tier` and `type` are, and a field a binary merely stops declaring is kept forever (D3);
the bound from D4: the only keys that pass through are those already in board.json, from another binary, a hand edit or a git merge of the records repository, and this binary adds, changes or merges none; a kept value has no size or shape limit and is trusted exactly as far as an entry's body is.

Tests, untagged with temp-dir fixtures:
- `keptValue`, in `store_test.go`, decodes the saved board.json as raw objects and returns the `json.Compact`ed value of `key` on the entry `slug` (the top level when `slug` is empty), and whether it is present.
- `TestLoad` gains `act func(*testing.T, *boardengine.Store)`, run after `check` and before the save, and `wantKeys`, a list of slug, key and compact value, an empty value meaning the key must be absent, asserted with `keptValue` after the save; a row with `act` or `wantKeys` saves, and the runner skips a nil `check`.
  Rows share one body: a top-level `"zz_top":{"k":[1,2]}`, a task `a` carrying `"zz_scalar":9007199254740993` and `"zz_nested":{"a":[1,{"b":"c"}]}`, a task `b` carrying `"zz_b":true`, a note `n` carrying `"zz_n":"x"` and a done task `fin`.
  One row per path asserts `a`'s two keys and `zz_top` with exactly those values: `SetStatus("a", …)`, `UpsertTask` of `a`'s title, `UpsertTasksBatch` changing `a`'s and `b`'s titles, `Promote("n")` (also asserting `zz_n`), `Prune()` and `SetDeps("a", []string{"b"})`.
  The set-status row also gains `saveOrder`, a list of whitespace-free substrings that must appear in that order in the compact saved file, set to the `"status":` member the row writes, then `"zz_nested"`, `"zz_scalar"`.
  `status` is `a`'s last known member after the write, since the seeded body gives `a` no short name, dates or run settings, so the order pins that kept keys come after every known field in sorted key order while the body lists them unsorted.
  The upsert and batch rows fail on today's float64 round trip, which writes 9007199254740992.
  A merge row runs `MergeTasks([]string{"b"}, {"slug":"a","title":…}, nil)` and asserts `a`'s keys kept and `zz_b` absent on `a` and at the top level, so the removed entry's key neither survives on the upserted entry nor moves elsewhere; a remove row runs `RemoveTask("b")` and asserts the same absences.
  A row on `{"version":1,"entries":[],"zz_top":"x"}` upserts a first entry and asserts `zz_top`.
  A case-variant row loads an entry carrying `"Title":"A"` and no `title`, checks the loaded title is `A`, and on save asserts the compact file holds `"title":"A"` and lacks `"Title"`.
  The existing tier-and-type row's body gains `"zz_scalar":9007199254740993` and a `wantKeys` entry for it, beside its existing `saveAbsent` of `"tier"` and `"type"`, so it pins that migration still drops the retired keys while keeping an unknown one.
  `TestLoad`'s doc comment names the kept keys.
- `addEntryKeys`, in `runlock_test.go`, splices the raw JSON members `members` into the entry `slug` of the board.json under `boardPath`, decoding and re-encoding with `json.RawMessage` so no value changes.
  `TestRunLock` rows gain `unknown string`; when set, the harness calls `addEntryKeys` on `locked` after seeding and before reading `before`, and an accepted row asserts with `keptValue` that the after file still carries each member.
  The existing accepted row "set-status to another value" sets `unknown` to `"zz_scalar":9007199254740993`, so it pins both the status write and the kept key passing the run lock; no row is added.

**ImpactSummary:** Every board write path inherits the kept keys through `Task`'s and the store file's JSON encoding; `ApplyPatch` and `NewTask` are the only round trips through JSON, every caller of `ApplyPatch` is in `store.go`, and boardengine's other consumers, such as `cleanupcli` and `battencli`, read known fields only.

**Commit:** 1: boardengine: keep board.json keys this binary does not know
