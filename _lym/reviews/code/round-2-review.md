---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/decision.go:883
    summary: add's new decision-lock refusal has no test that reaches it, which PATTERN-refusal-way-forward requires
    fix: >-
      In internal/loomcli/decision_test.go add a row to the decisionVerb refusal table
      (the table beside TestDecisionVerb_AppendsAndCommits, or a new row set in TestDecisionVerb)
      that sets f.lockErr to an error reading "take the run's decision lock, boom" on a fake past its approval
      and calls decisionVerb with goodDecisionInput().
      Assert exit non-zero, the message contains "decision lock", "way forward" and "re-run the verb; nothing was written",
      that f.appends == 0 and f.commits == 0, and that the record file is unchanged.
      Do not add a separate top-level test; the row belongs to the add refusal table.
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/decision.go:1334
    summary: decisionReplaceVerb's refusals for a failed record write, and for a failed restore after it, have no test that reaches them
    fix: >-
      In internal/loomcli/decision_test.go give decisionFake two fields,
      writeFirstErr (fails only the first writeRecord call) and writeAllErr (fails every writeRecord call),
      and honour them in the writeRecord closure before os.WriteFile (the closure already counts f.recordWrites).
      In internal/loomcli/decisionreplace_test.go add two rows to TestDecisionReplaceVerb_Refusals.
      Row 1, WriteFailsAndTheRestoreSucceeds, sets writeFirstErr
      and wants "write the decision record", "way forward" and the re-run of "lym loom decision replace".
      Row 2, WriteAndRestoreBothFail, sets writeAllErr
      and wants "restoring it failed too" and "way forward: rewrite the file named above".
      The table's shared assertions then check that nothing committed, stored or posted, and that the lock was released.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/boardengine/runlock.go:67
    summary: Doc comments in boardengine say decision add alone, or the one way, changes a beam project record, which the diff makes false by adding replace
    fix: >-
      In internal/boardengine/runlock.go line 67 reword the sentence that says `lym loom decision add <slug>` alone changes the record after that
      to say that only `lym loom decision add` or `lym loom decision replace` on the beam's slug changes the record after that.
      In internal/boardengine/doc.go line 472 reword "the one way the project's scope changes after that" the same way,
      so it no longer calls add the only way.
      The refusal message itself may keep naming add, which stays a valid way forward.
---

### [BLOCKING] add's decision-lock refusal is never reached by a test

**Location:** `internal/loomcli/decision.go:883`
**Issue:** The diff makes `decisionVerb` take the per-run decision lock and refuse with `decisionWayLock` when it cannot.
That is a new refusal.
`PATTERN-refusal-way-forward` requires that every new loom refusal's message names the way forward and that a test reaches it, and the decision record's Constraints repeat this for every new refusal of D1 and D3.
The replace verb has a `LockFails` row (`decisionreplace_test.go:264`) and the store verb has a `LockFails_RefusesWithTheRetryWay` row.
A search of the test files for `lockErr` finds only those two users and the fake itself, so no test drives `add` into the lock failure.
`TestDecisionLock_SerializesRecordWriters` reaches add's lock only on the success path.

### [BLOCKING] replace's failed-write refusals are never reached by a test

**Location:** `internal/loomcli/decision.go:1334`
**Issue:** When `deps.writeRecord(file)` fails, `decisionReplaceVerb` tries to write the prior bytes back.
It then returns one of two refusals: "write the decision record: …; restoring it failed too: …" with `decisionWayRestoreRecords`, or "write the decision record: …" with `replaceWayRetry`.
The first failed write is a raw I/O error, but the restore-failed branch is a refusal of its own with a distinct way forward.
`TestDecisionReplaceVerb_CommitOutcomes` reaches the restore failure only after a commit error, through `landReplace`.
The fake's `writeRecord` fails only from the second write on (`restoreErr`), so no test can make the first write fail.
Neither refusal is reached.

### [LOW] Doc comments say add alone changes a beam project record

**Location:** `internal/boardengine/runlock.go:67`
**Issue:** `ProjectRecordHeld`'s doc says "`lym loom decision add <slug>` alone changes the record after that".
The `internal/boardengine/doc.go` comment at line 472 calls `decision add` "the one way the project's scope changes".
After this diff `lym loom decision replace <beam-slug>` changes the project record too.
The diff already updated the pawl recipe, the project-record stencil, the overview and `PATTERN-run-records` to name both verbs, so these two comments are the ones left behind.

## Focus verdicts

- **`TestDecisionReplaceVerb_Refusals` and `_CommitOutcomes` cover every refusal of `decisionReplaceVerb` and `landReplace`.**
  Every one has a row that asserts its message and way forward, except two: the failed-write pair filed as F2, and the propagated read failures of the status, recipe and record reads, which are raw I/O.
  The `--from` and `--title` empty checks share one code site with the `EmptyBase` row.
  The raddle filing rows, the Discussion-approval row, both Webster-span rows, the caller rows and every `landReplace` outcome are reached.
- **`TestDecisionCalls_RefreshEveryStoredRecord` has no beam row, and that is not a defect.**
  The `record.project` exclusion sits in `boardRecordStoredOf`, which the `BeamProjectRecordWhoseEntryHoldsOne` row of `TestBoardRecordStoredOf` reaches and which fails if the exclusion breaks.
  The composition through `boardCopyDue` is covered by the `AddOnARunWithNoStoredRecord` row.
  Only the one-line wiring of `boardRecordStoredOf(record, …)` in `preRun` is untested, and a test for it would need a forged hub for no further behavior.
- **The `store` help states both points.**
  It says it works on a recipe with no Discussion approval when the entry carries a stored record, and refuses a raddle run whose filing is done.
  `decisionStoreVerb` matches it, including the no-stored-record refusal, which the `DarnRun_RefusedNamingRecordPath` row reaches.
- **`contracts/specs/loom-discussion-spec.md` names both verbs and when each applies.**
  It names `replace` for changing, withdrawing and folding, `add` for a new call, and `lym board upsert --record-file` for a Sketch's stored record once a motif run has made it.
  It also states the history cut.
- **`CheckAddedEntriesSubset` fails a repeated entry.**
  It counts each current entry once, so a file repeating an entry fails, and the `the file repeats an entry` row asserts it.
  The comparison trims trailing whitespace only at an entry's end and compares every other byte exactly, so it is slightly looser than the record's "byte for byte".
  The `replace` help text states the tolerance, and it only lets whitespace after an entry's last line differ, so it is not a defect.
- **The beam project-record texts name both verbs.**
  `docs/overview.md`, `loom-directive-project-record.md` and `pawl-recipe.yaml` all name `replace` beside `add`.
  The only leftovers are the two boardengine comments in F3.
- **`TestDecisionLock_SerializesRecordWriters` pins add and carry-over serialization behind a replace's lock, and `TestDecisionStoreVerb_TakesTheDecisionLock` pins store's take, release and refusal.**
  The remaining gap is add's lock-failure refusal, filed as F1.
- **`TestRecordHistoryCut` reaches the whole-history fallback through `recordHistoryFrom`.**
  That is the function `recordHistory` calls, not an inline copy.
  The `ATrailerWhoseSumMatchesNoEarlierVersion_CutsNothing` and `ATrailerNamingAnotherRecord_CutsNothing` rows assert the fallback, and `TestRecordHistory_CutsAtTheNewestReplace` drives `recordHistory` itself over real commits.
