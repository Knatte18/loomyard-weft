# Decision record — board-field-drop

## Goal

A `board.json` rewrite by a `lyx` binary keeps every entry key and top-level key that binary does not know, so a deploy that adds a board field stops losing that field's data to a long-lived run from an older binary that writes status.
The open items from the `lyx cleanup` PR (#574) are closed: the `internal/lyxtemp` concurrent-sweep race is fixed and pinned by tests, and the other three items are confirmed already met.

## Scope

In:

- `internal/boardengine`: board.json load and save keep unknown keys, per entry and at the top level, across every write path.
- `internal/boardengine/doc.go`: a section on unknown keys, the D5 bound and the D3 retirement rule.
- `internal/boardcli/doc.go`: a line that `get` and `list-full` print an entry's kept keys.
- `internal/lyxtemp`: the sweep's removal retry is bounded by time instead of a fixed count, with a seam on `Root` that tests drive through `Sweep`.

Out:

- Other record files a long-lived process rewrites whole, such as shed `status.json` and `seed.json`: the board entry's title and Idea name board writers only (see Open risks).
- A schema-version refusal for board writers (D6).
- Restoring the `created`/`revised` dates the incident already dropped: a backfill rerun is operator action after this deploys.

Already met, no card (evidence in the support log):

- `lyx cleanup`'s refusal outside the prime is pinned by `internal/cleanupcli/cleanup_integration_test.go`.
- `lyx batten cleanup`'s skip of a deleted run directory without a committed `status.json` is pinned by a row of `internal/battencli/cleanup_test.go`.
- The `testtiming` "resign arrow" item was a plan-card finding of `test-tmp-cleanup`; the landed code and `cmd/testtiming` docs already match.

## Decisions

### D1 — An entry keeps the keys this binary does not know

Decision: decoding a board.json entry collects every key the struct decode does not consume, with its raw JSON value, onto the in-memory `Task`; encoding a `Task` writes them back after the known fields, in sorted order.
A key counts as known exactly when `encoding/json` would decode it into a field, including its case-insensitive match, so a case-variant such as `"Status"` lands in the known field and is never also kept.
A value is kept as raw JSON, never interpreted.
Rationale: the loss comes from decoding into a fixed struct; keeping the remainder on the record makes every write path safe at once, since each goes through load → mutate → save.
`ApplyPatch` and `NewTask` round-trip through JSON, so the kept keys are part of `Task`'s JSON encoding.

### D2 — board.json keeps its unknown top-level keys

Decision: the store file's unknown top-level keys are kept the same way and written back on save.
Rationale: a future store-level field beside `version`, `entries` and `legacy_done` is lost the same way.

### D3 — Retired keys are consumed explicitly

Decision: `tier` and `type` stay known keys of the decode shape, consumed by the in-memory migration and never kept, so the saved entry carries neither, as today.
From this change on, retiring a board field means naming it in the decode shape so it is consumed and dropped; a field a binary merely stops declaring is kept forever.
`doc.go` states this rule.
Rationale: keeping retired keys would undo the migration's drop on first write.

### D4 — A kept key is data this binary cannot change

Decision: upsert payload validation is unchanged: a key outside `UpsertPayloadKeys` is still refused, so no caller adds or changes an unknown key through this binary.
Kept keys travel with their entry: set-status, upsert, batch upsert, promote, prune and set-deps keep them; `remove` drops them with the entry; a merge's upserted entry keeps its own, and each removed entry's go with it.
The run lock does not compare kept keys.
Kept keys appear only where a `Task` is printed whole, `lyx board get` and `lyx board list-full`; `lyx board list`, the README and any rendered view use known fields.
Bound: the only keys that pass through are those already in board.json, written by another binary; this binary adds, changes or merges none.

### D5 — The guarantee starts with this binary

Decision: `doc.go` states that a writer keeps unknown keys from this binary onward, that a process started from an older binary still drops them, and that a kept key is carried unchanged while known fields beside it change.
Rationale: an old binary cannot be fixed after the fact, so the first deploy carrying this change still meets old long-lived processes.

### D6 — No schema-version refusal

Decision: the store's `version` check stays as it is and is not bumped; a board with a newer field is written, not refused.
Rationale: a refusal stops a long run's status writes, which halts the run, and needs a version bump with every field addition.

### D7 — The sweep's removal retries for a time budget

Decision: every `lyx` process that creates a run directory sweeps the root, so many sweepers can run at once, and each one's lock probe can recreate a dead directory's lock file once while another sweeper removes it.
The fixed three attempts 5 ms apart cover one recreation reliably but not many, and a sweep failure fails `lyx cleanup`'s temp segment.
`removeRunDirectory` keeps retrying a failed removal while the directory exists, until a total time budget of one second runs out, then reports the last error.
`Root` carries the removal step and the clock-and-sleep the retry uses as unexported fields, defaulting to `os.RemoveAll` and the real clock, so tests inject them and drive `Root.Sweep` without real sleeping.
Bound: the budget caps one directory's retries at one second; the seam is unexported, uses only the standard library, and stays within `internal/lyxtemp`'s leaf import set.

## Constraints

- `storedEntry` embeds `Task`: any custom JSON decoding on `Task` is promoted onto `storedEntry`, so the decoding must keep D3's consumption of `tier` and `type` working.
- `PATTERN-test-economy` and `PATTERN-test-speed`: board cases extend existing table tests in `internal/boardengine` where one covers the write path, untagged, with temp-dir fixtures; lyxtemp cases go through `Root.Sweep` with injected removal and clock.
- `PATTERN-lyx-temp-root`: lyxtemp test directories live under the test root, as the existing tests do.
- A board.json written by this binary stays readable by older binaries: kept keys are ordinary JSON object members, and no known key changes shape.

## Auto-mode assumptions

- Key-order churn when an old and a new binary alternate writes is harmless.
- One second suffices as the removal budget: each recreation is a single probe by a single sweeper.
- Every other self-pick is recorded in the support log's Question ledger.

## Open risks

- Shed `status.json` and other whole-file records rewritten by a run's long-lived process and by short-lived processes from a newer binary in the same run can lose fields the same way; no board entry tracks this.
- A kept key goes stale when it depends on known fields an older binary changes, such as a future field derived from scope or merged by a rule; a merge keeps only the upserted entry's own kept keys.
- A newer binary retiring a field this binary still knows sees this binary write it back.
- On Windows a held lock file cannot be deleted, so the retry's behavior there is unverified; no board entry tracks it.

## Acceptance criteria

- A board.json whose entry carries an unknown scalar key and an unknown nested-object key, and whose top level carries an unknown key, keeps all three with equal JSON values after each of: set-status, an upsert of a known field, a batch upsert, promote, prune and set-deps.
- A board.json with no entries and an unknown top-level key keeps that key across the upsert that creates the first entry.
- A merge keeps the upserted entry's unknown keys and drops each removed entry's; `remove` drops the removed entry's.
- A status write on a run-held entry that carries unknown keys passes the run lock.
- An entry key that differs from a known field only in case is decoded into that field and not kept, so the next save carries the field once.
- An upsert payload carrying an unknown key is still refused with the existing message.
- An entry in the old `tier`/`type` shape still migrates, and the saved entry carries neither key.
- `lyx board get` and `lyx board list-full` print an entry's unknown key.
- A sweep removal that fails several more times than three, then succeeds within the budget, removes the directory with no failure reported.
- A removal that keeps failing past the budget is reported as a sweep failure and leaves the directory in place.
- A path that vanishes between attempts counts as removed.
- `internal/boardengine/doc.go` describes unknown-key keeping, the D5 bound and the D3 retirement rule; `internal/boardcli/doc.go` says `get` and `list-full` print kept keys.

## Notes for the plan writer

- Load and save live in `internal/boardengine/store.go` (`loadedStoreFile`, `storeFile`), the entry shape in `task.go`, and `storedEntry` in `migrate.go`.
- `checkRunLocks`/`scopeChanged` in `runlock.go` compare named fields only.
- The sweep retry is `removeRunDirectory` in `internal/lyxtemp/sweep.go`; `Root` already carries an injected `now`, and the concurrent subtest is in `sweep_test.go`.
