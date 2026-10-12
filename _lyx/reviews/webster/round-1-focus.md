---
round: 1
focus:
  - "internal/boardengine/task.go and store.go: does every write path (set-status, upsert, batch upsert, promote, prune, set-deps, merge, remove) keep an entry's and the store's unknown keys with exact raw digits, and does `ApplyPatch` never round-trip a kept value through `float64`?"
  - "internal/boardengine/migrate.go: does `storedEntry.UnmarshalJSON` consume `tier` and `type` case-insensitively and keep every other unknown key; is `Task.MarshalJSON` promotion onto `TaskWithLayer` harmless as the plan states?"
  - "internal/boardengine/store.go `Load` and the follow-up fix commit: is the kept-key decoding kept outside the shared lock, and does `Load` still reset `unknown` on every path?"
  - "internal/boardengine `cloneTasks`, `checkRunLocks`, `scopeChanged`: does the copy alias no `Unknown` map, and is a kept key never compared by the run lock?"
  - "internal/lyxtemp/sweep.go: does the retry cover only the not-empty symptom, with one injected-clock budget per `Root.Sweep` call shared across removals, and do `ProcessRoot` and `openRoot` keep real `os.RemoveAll`, `time.Now` and `time.Sleep`?"
  - "Test economy: do the new `TestLoad`, `TestRunLock`, `stepLookupContract` and `TestSweep` rows each cover distinct behavior, and does any added behavior lack a row?"
  - "Per-card mechanical check: for each card's Create, Edit and Delete groups, did the type-specific check run against every target (for example `assert-no-callers` for the deleted `removeAttempts`)?"
---

Round 1 seed: the diff spans three cards (boardengine kept keys, boardcli pinning rows, lyxtemp sweep budget) plus a late fix commit on `Load`.
Entries name where to look; none states a verdict.
