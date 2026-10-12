---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D1, third sentence of Decision)
    summary: D1 requires a kept value to survive every write path byte-for-byte, which the indented, HTML-escaping board.json encoder cannot deliver; the acceptance criteria rightly ask for equal JSON values
    fix: In D1 replace "survives every write path byte-for-byte, `ApplyPatch` included" with "survives every write path as the same JSON value, `ApplyPatch` included; the store's indented encoding may re-indent a nested value and escape a string's characters, but never changes the value". Keep the rest of the sentence on number digits unchanged.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D7 Bound); _lyx/discussion/support-log.md (Q7)
    summary: D7's Bound says the one-second cost arises only while concurrent sweepers recreate lock files, but the directory-not-empty test also matches any process still writing into a dead owner's directory, which costs every newRun up to the full second while it writes
    fix: In D7's Bound replace "reached only while concurrent sweepers keep recreating lock files" with "reached while any process keeps adding entries to a dead directory, a concurrent sweeper's lock probe or a process outliving the directory's dead owner, and paid by every `newRun` for as long as that lasts". In Q7 change "so a persistent non-race failure does not cost every `newRun` a second" to "so a persistent failure of another kind does not cost every `newRun` a second; a persistent writer into a dead directory still does".
---

### [MEDIUM] D1's byte-for-byte requirement is unattainable and contradicts the acceptance criteria

**Location:** `_lyx/discussion/decision-record.md`, D1.

**Issue:** board.json is written by `state.WriteJSON` (`internal/state/state.go:48`), which calls `json.MarshalIndent`.
`encoding/json` writes a `json.RawMessage` through `appendCompact(out, b, opts.escapeHTML)` (`encode.go`, `marshalerEncoder`), and `MarshalIndent` then re-indents the result.
A kept nested object therefore comes back re-indented, and a kept string holding `<`, `>` or `&` from a hand edit comes back as `<` and its kin.
Digits survive compaction, so the integer-above-2^53 requirement holds; the byte-for-byte wording does not.
The acceptance criteria ask for "equal JSON values", which is the attainable contract.
A plan writer reading D1 literally would either build a byte-preserving writer outside `state.WriteJSON` or write a byte-comparing test that fails.

### [LOW] D7's Bound misattributes the one-second path

**Location:** `_lyx/discussion/decision-record.md`, D7 Bound; `_lyx/discussion/support-log.md`, Q7.

**Issue:** D7 retries on a directory-not-empty error from removing the directory itself.
In `os.RemoveAll` (`removeall_at.go`) that error surfaces only when the directory's children were all removed and the final `unlinkat` still finds an entry, so any process adding entries to the directory during removal triggers it, not only a peer's lock probe.
A process that outlives a killed owner, for example a child still writing into the owner's run directory, does not hold the owner's lock (`gofrs/flock` opens the lock file through `os.OpenFile`, close-on-exec), so the directory reads as dead and every sweep spends its whole budget on it.
The numeric bound, at most one second per `newRun`, stays true and the acceptance rows on the shared budget pin it; the Bound's condition and Q7's claim that no non-race failure costs every `newRun` a second are false for this case.

## Focus verdicts

- Kept-key byte fidelity: D1 requires the exact value across every write path, `ApplyPatch` included, and the first acceptance criterion pins an integer above 2^53 on the upsert and batch-upsert rows.
  `Store.MergeTasks` (`internal/boardengine/store.go:822`) reaches the same `ApplyPatch` for an existing slug and then `UpsertTask`, so the upsert rows pin the shared code; no separate merge digit row is needed.
  Not a defect beyond F1's wording.
- Kept-key output surface: `outputSuccessWithTask` is called by promote (`internal/boardcli/cli.go:145`), upsert (`:396`) and merge (`:752`); `get` and `list-full` print whole `Task`s through `outputGetTask` and `outputListFull`; upsert-batch prints a count.
  D4, the Scope line for `internal/boardcli/doc.go`, the acceptance criterion and Q6 all name `get`, `list-full` and the envelope of `upsert`, `promote` and `merge`.
  Not a defect.
- Sweep cost bound: a failure other than the symptom costs one removal attempt per dead directory, stated in D7's Bound and pinned by the "any other error is reported after one attempt" criterion.
  The one gap is the condition under which the symptom path is reached, raised as F2.
- Retry clock: D7 states the retry clock-and-sleep pair is separate from the creation-grace `now`, and that the injected sleep advances the injected retry clock; the Notes repeat why (`fixedClock` freezes `now`).
  Not a defect.
