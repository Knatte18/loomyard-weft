---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/alignment/records.go:88
    summary: The parent's alignment attempt.json is read-modified-written through os.ReadFile and os.WriteFile, so every rewrite drops keys a newer binary wrote, which PATTERN-tolerant-records forbids
    fix: In internal/alignment/records.go, replace the json.MarshalIndent plus os.MkdirAll plus os.WriteFile body of WriteAttempt with internal/state, so a rewrite keeps the keys the Attempt type does not know. Use state.UpdateJSON with a lock path under a.Dir for the Close and MarkApplied read-modify-write sites in session.go (lines 346-371), and state.WriteJSON for the first writes in Begin and failBegin. Read attempt.json through state.ReadJSONNoMkdir (or state.ReadJSON) in readAttemptFile instead of os.ReadFile plus json.Unmarshal. Add one row to the existing alignment records test that round-trips an attempt.json carrying an unknown top-level key through Close or MarkApplied and asserts the key survives.
---

### [BLOCKING] alignment attempt.json rewrite bypasses internal/state and drops unknown keys

**Location:** internal/alignment/records.go:88 (WriteAttempt), internal/alignment/records.go:147 (readAttemptFile), internal/alignment/session.go:346-371 (Close, MarkApplied)

**Issue:** The task added `PATTERN-tolerant-records` (D9).
It requires that a rewrite of a Go-written JSON record goes through `internal/state`, which keeps the keys the record's type does not declare, or through a type that keeps them itself.
The parent's new `internal/alignment` package arrived in the same merge and was written before that rule existed.
`Session.Close` and `Session.MarkApplied` read `attempt.json` with `ReadAttempt`, change `Closed`, `ClosedReason`, `Applied` or `Notified`, and write the whole struct back with `WriteAttempt`.
`WriteAttempt` encodes with `json.MarshalIndent` and writes with `os.WriteFile`, so a field a newer binary added to `attempt.json` is dropped by the first rewrite from an older long-lived loom process.
That is the loss this task exists to stop.
The record is also durable run state that the doc says a restarted driver reads to reuse an attempt under its idempotence key, so a dropped key is a lost record field.

The enforcement scan only fails on `DisallowUnknownFields`, so verify passed on the merged tree; the rule is still broken, and the interaction of the two sides is the defect.
The new readers themselves are tolerant, as are the other parent records: `parentreview`'s store goes through `state.UpdateJSON` and `state.WriteJSON`, `readHeldJSON` is a read-only decode, and the `AlignmentNote` and approval records are written once whole, not rewritten.
Only `attempt.json` is rewritten outside `state`.
`attempt.json` is also read and written with no lock, while `watchRun` and the driver can touch the same attempt; routing it through `state` adds the write lock the pattern's `UpdateJSON` primitive provides.

No other interaction defect turned up in the seams the focus file names:

- The renamed `state.ReadJSONNoMkdir` is the call the parent's `loomcli` sites already use.
- The `shedrun` `AlignmentDir` and `AlignmentRel` additions leave the seed decode alone.
- The `websterengine` doc edits from the two sides describe different sections.
- The `test-budget.yaml` rows merged without a conflict.
- The parent adds no ticker or sub-second wait outside the declared list.
