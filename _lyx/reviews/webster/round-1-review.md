---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/boardengine/task.go:94
    summary: eachMember and skipValue hand-scan kept values, but no row keeps a string holding an escaped quote, a brace or bracket, a comma, or a key with an escape
    fix: In internal/boardengine/store_test.go extend keptBoardBody and keptA so task a also carries a kept key whose value is a string such as "q\"}],x" and a nested object holding such a string, and a kept key spelled with an escape such as "zz_esc"; add them to keptA so every TestLoad row asserts their compact values after the save. No new top-level test.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/boardengine/store.go:85
    summary: loadedStoreFile.UnmarshalJSON and eachMember comments say the decode runs while the shared lock is held, but the follow-up commit moved it after the lock is released
    fix: In store.go reword the second line of loadedStoreFile.UnmarshalJSON's comment to drop the claim about the shared lock, and in task.go reword eachMember's comment to drop "which keeps a load under the shared lock cheap"; keep only why the walk is a single byte pass if that still matters.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/boardengine/migrate.go:18
    summary: storedEntry still has Task.UnmarshalJSON promoted onto it and has no UnmarshalJSON of its own, so json.Unmarshal into a storedEntry silently drops tier and type; card 1 specified storedEntry.UnmarshalJSON
    fix: In migrate.go add `func (e *storedEntry) UnmarshalJSON(data []byte) error` that decodes into a decodedEntry and sets *e = storedEntryFrom(decoded, data), then trim storedEntry's comment accordingly; loadedStoreFile.UnmarshalJSON keeps its single-pass path.
---

### [MEDIUM] Hand-written JSON scanner is only exercised on simple values

**Location:** internal/boardengine/task.go:94
**Issue:** The fix commit replaced the plan's `splitUnknown` (a `json.Unmarshal` into raw members) with `eachMember`, `eachElement`, `skipSpace` and `skipValue`, a byte scanner that trusts its input.
Its string branch (escape skipping, a closing quote inside a nested value) and its key-unescape branch are the places a scanner like this goes wrong, and every kept value in `TestLoad`, `TestRunLock` and the boardcli rows is a number, a bool, a plain string or a plain nested object.
A kept string holding an escaped quote, `}`, `]` or `,`, and a key written with an escape, are added behavior with no covering row.
The tests pass and the code reads correctly on inspection, so this is a coverage gap, not a found defect.

### [LOW] Stale comments claim the decode happens under the shared lock

**Location:** internal/boardengine/store.go:85
**Issue:** `Store.Load` now reads raw bytes under the lock and decodes after releasing it, which is the point of the fix commit.
`loadedStoreFile.UnmarshalJSON` still says the single-pass design keeps the load cheap "while the shared lock is held", and `eachMember` says the walk keeps "a load under the shared lock cheap".
Both describe a state the code no longer has.

### [LOW] storedEntry silently drops tier and type if decoded directly

**Location:** internal/boardengine/migrate.go:18
**Issue:** Card 1 gave `storedEntry` its own `UnmarshalJSON` so the promoted `Task.UnmarshalJSON` could not shadow `Tier` and `Type`.
The implementation instead routes the load through `decodedEntry` and `storedEntryFrom`, and `storedEntry` still carries the promoted method.
Today only `loadedStoreFile.UnmarshalJSON` fills one, so nothing misbehaves, but `loadedStoreFile.Entries` is still declared `[]storedEntry` and a future direct decode loses the retired keys with no error.

## Checked and found sound

- Every write path keeps kept keys: all are load, mutate, save over `Task` values, and `ApplyPatch` overlays on `map[string]json.RawMessage`, so no kept value passes through `float64`.
  `TestLoad` rows cover set-status, upsert, batch upsert, promote, prune, set-deps, merge and remove.
- `Store.Load` resets `unknown` on every path, and the kept-key decode now runs after `ReadJSON` releases the shared lock.
- `cloneTasks` clones `Unknown`; `checkRunLocks` and `scopeChanged` never compare it; the `TestRunLock` accepted row pins that.
- `TaskWithLayer` is not JSON-encoded anywhere, so the promoted `MarshalJSON` is harmless.
- `lyxtemp`: `isRemovalSymptom` matches only a `*fs.PathError` on the path itself with `fs.ErrExist`; one `deadline` per `Sweep` is shared across removals; `openRoot` sets `os.RemoveAll`, `time.Now` and `time.Sleep`; no `removeAttempts` reference remains (`grep` over `.go` and `.md`).
  The five sweep rows each cover distinct behavior.
- The new `stepLookupContract` rows each cover a distinct verb's printed envelope.
- `lyx gate test ./internal/boardengine/... ./internal/boardcli/... ./internal/lyxtemp/...` is green.
