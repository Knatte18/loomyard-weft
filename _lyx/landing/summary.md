# boardengine: keep unknown board keys; bound lyxtemp sweep retry

A `board.json` rewrite by a `lyx` binary now keeps every entry key and top-level key that binary does not know.
Before, a deploy that added a board field (such as `created`/`revised`) lost that field's data whenever a long-lived `lyx batten run` or `lyx shed step` process from an older binary wrote status.

## What changes

- `internal/boardengine`: decoding an entry collects every key the struct decode does not consume, as raw JSON, onto `Task`; encoding writes them back after the known fields in sorted order.
  The store file keeps unknown top-level keys the same way.
  Every write path (set-status, upsert, batch upsert, promote, prune, set-deps, merge) keeps them, and `remove` drops them with the entry.
- `lyx board get`, `list-full` and the task envelope of `upsert`, `promote` and `merge` print an entry's kept keys.
- `internal/lyxtemp`: a sweep's removal retry is limited to the lock-file recreation symptom (directory-not-empty from removing the directory) and bounded by a one-second time budget shared by one `Root.Sweep` call, instead of three fixed attempts.
  This closes the concurrent-sweep race behind the flaky `TestSweep/two_concurrent_sweeps_both_succeed`.
  `Root` carries the removal step and a retry clock and sleep as unexported fields, so tests drive `Sweep` without real sleeping.
- The other `lyx cleanup` follow-ups (refusal outside the prime, `batten cleanup` skipping a deleted run directory, the `testtiming` doc) were already met and pinned by existing tests and docs; no change.

## Design decisions

- A key counts as known exactly when `encoding/json` would decode it into a field, so a case-variant such as `"Status"` lands in the known field and is not also kept.
- Kept values are never decoded into a Go type, so a number keeps its exact digits (an integer above 2^53 survives `ApplyPatch`).
  The store's indented encoding may re-indent a nested value or escape a string's characters, but the JSON value is unchanged; byte-for-byte equality is not promised.
- `tier` and `type` stay consumed by the in-memory migration and are never kept.
  Retiring a board field from now on means naming it in the decode shape so it is dropped; a field a binary merely stops declaring is kept forever.
  `internal/boardengine/doc.go` states this.
- Upsert payload validation is unchanged: a key outside `UpsertPayloadKeys` is still refused, so this binary adds or changes no unknown key.
  The run lock does not compare kept keys.
- The guarantee starts with this binary: a process started from an older binary still drops unknown keys.
- No schema-version refusal: a refusal would halt a long run's status writes and need a version bump per field.
- `Load` reads the raw bytes under the shared lock and decodes after releasing it.
  Decoding under the lock starved the writer in `boardtest`'s `TestConcurrentReadsDuringUpserts`.
- The sweep retry adds at most one second to one `newRun`, only while something keeps adding entries to a dead directory.

## Not covered

- Other whole-file records a long-lived process rewrites, such as shed `status.json` and `seed.json`, can lose fields the same way.
- Restoring the `created`/`revised` dates already dropped is a backfill rerun by the operator after this deploys.

## To check by hand

- `internal/boardengine/task.go`: the kept-value scanner is tested against escaped quotes, brackets and an escaped key; confirm no other string shape (for example unicode escapes) mis-scans.
- `internal/boardengine` doc and `Task` encoding: the design wording promises equal JSON values, not equal bytes; confirm that matches what the tests pin.
- `internal/lyxtemp/sweep.go`: on Windows a held lock file cannot be deleted, so the retry's behavior there is unverified.
- Not run: `go vet` under the tag runs, and tests of other `boardengine.Task` consumers such as `cleanupcli` and `battencli`.
