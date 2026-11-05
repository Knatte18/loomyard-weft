```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each function and error path the diff changes with concrete inputs.

## Per function: case run, result

- `raddleFile.Call`, store done (`StoreDone` true), interactive, no approval: skips `verify` and goes to `finish`. Correct, since the store marker overrides the approval check.
- `raddleFile.Call`, crash after step 1 (hash file written): `stage` skips on the hash match. Correct.
- `raddleFile.Call`, crash mid step 2: `writeRecord` skips only when both the hash file and the record exist, and `writeRecords` writes the hash file last. Correct.
- `raddleFile.Call`, crash after `File` saved `board.json` but before `writeRaddleStore` completes: **not correct**, see finding 1.
- `raddleFile.Call`, crash after step 3 with a successful store write: retry finds `StoreDone` true, skips the checks and re-run steps 1 to 3, and goes to steps 4 to 7. Correct.
- `raddleFile.Call`, steps 4 to 7 on retry: `CreateBeam` and `Release` are idempotent, `ReleaseTasks` skips absent, done or non-draft parts, and `NoticeSent` gates step 6. Correct.
- `raddleFile.Call`, step 6: a crash between `Notify` and the `NoticeSent` update sends a second notice. This is an accepted duplicate, not raised as a finding.
- `raddleFile.verify`, interactive mode with a nil approval or a hash mismatch: returns a reason string and no error. Correct.
- `raddleFile.verify`, `Recheck` landed entries: passed to the notice. Correct.
- `raddleFile.sendBack`, `Returns` 0, 1, 2:
  - Returns 0 and 1 return to Raddle-Write and become 1 and 2.
  - Returns 2 halts.
  - This matches "2 returns, then halt".
  - The halt path leaves the stage in place and the count unreset. Correct.
- `raddleFile.sendBack`, state update: `Approval` is cleared in the same update that raises `Returns`. Correct.
- `FileRaddle`, store already holds the same hash and `created`: returns `noWrite`. Correct.
- `FileRaddle`, `Check` refusal: error wraps `ErrRaddleRecheck`, `saved` is false, so the record files are removed. Correct.
- `FileRaddle`, dependency on an entry that is gone from the board or is the run's entry: dropped. Correct.
- `FileRaddle`, dependency on a part filed earlier in the same write: kept, because `dependencyOrder` orders parts first and `store.index` finds them. Correct.
- `FileRaddle`, store write fails after the save: **not correct**, see finding 1.
- `fileRaddleEntry`, `Discussed` set after `UpsertTask`: set in the same write. Correct.
- `ReleaseTasks` and `EndRaddleEntry`, absent or done entries: write nothing. Correct.
- `raddleWrite.Call`, first call:
  - `recorded.Mode` is empty, so `AutoApprove` is read and the mode is stored once.
  - Later calls read the stored mode, not the key. Correct.
- `raddleWrite.Call`, fresh entry (return or goto):
  - `clearLastAttempt` ignores `ErrNotExist`.
  - The position is recorded after the clear, so a crash between the two is retried safely.
  - Interactive mode also clears the approval. Correct.
- `raddleWriteEntry` table closure: reads `Mode` and `ReturnReason` per `Call`, after `raddleWrite.Call` has recorded the mode. Correct.
- `raddleProposeVerb`, first passing call:
  - `StoreColdProposal` is a no-op once `cold_stored` is set.
  - `NoticeSession < Session` gates the notice.
  - Correct, except for finding 2 on the notice-failure path.
- `raddleApproveVerb`:
  - Refuses with `ProposedSession` 0 or below `Session`.
  - Re-runs the checks and the re-check.
  - Writes the state approval, then the approval file.
  - Correct, except for finding 2.
- `raddleparser.Check.coverage`: exact-text bullet matching, the PATTERN carrier count (0 gives uncarried, more than 1 gives shared), and unknown ids. Correct.
- `raddleparser.Check.criteriaNameDecisions`: `\bD[1-9][0-9]*\b` matches `D10` whole, not `D1`. The path check uses `reachable`, which has a `seen` guard against cycles. Correct.
- `raddleparser.Check.dependencies`: self-dependency and cycles are found via the `onStack` state. Correct.
- `raddleparser.Check.sharedPackages`: skipped when a path exists in either direction. Correct.
- `raddleparser.Check.shape`: the tapestry `fallthrough` into the tasks case still checks that no project is present. Correct.
- `raddleparser.Check.vocabulary` and `slugs`: project `type` is checked, the Sketch type is refused, and the Sketch's slug and the run's own entry are exempt from `takenSlug`. Correct.
- `RecheckBoard`: done or gone entries land without failing, a moved `revised` stamp fails, and an unchecked open entry sharing a module label fails. Correct.
- `checkNewSlugs`, live Sketch with an equal `created` stamp: exempt. A promoted note with a different `created`: refused with the rename way forward. The store is read before the write. Correct.
- `checkDiscussedPromotion` and `promoteTo`: restores the label and releases the task only for recipe `raddle`, and refuses any other recipe with the way forward. Correct.
- `checkSketchLabel`: a legacy task that already carries the label keeps taking writes. Correct.
- `FabricReplay.Apply`, store files:
  - Set aside when a filed slug's add was refused or lost, or when `store.json` is undecodable.
  - `AtomicWrite` with a nested relative path does create the directories.
  - Correct.
- `raddleStartRefusal`: no record, another Sketch's slug, a filed slug, or another `created` is refused. The own-`created` case passes. Correct.
- `BeamUp.handOff`:
  - Writes the marker after the mail.
  - `posted` false halts with no marker.
  - No proposal writes the marker with no mail.
  - Correct.
- `RecordSessionUsage` and `fillFromTranscripts`:
  - A locate or read failure, a missing transcript, or no usage falls back to `SessionUsage` counts with unknown cost, logged at Warn.
  - Zero-token messages cost nothing.
  - An unpriced model leaves the cost nil.
  - Correct.
- `tallyTranscripts`, forks: each message is priced at its own model, forks included. Correct.
- `rolloutTotalUsage`: input is `Input - CachedIn`, cache-write is zero when absent, and output is not double-counted. Correct for the D12 split.
- `Runner.finalize` and `reportStoppedSession`: a usage read and a `SessionEnd` at every outcome and on stop. A session with no id is skipped. Correct.
- `Finalize.recordLandedSize`: the base is the parent head kept right before the merge, and the second `mergeAndGate` pass re-keeps it. A resume past the merge records none and logs Info. Correct.
- `recordWrite.Call`, stored record present: writes the record verbatim. Correct. For the log, see finding 3.
- `SplicePart` and `listedDecisions`: a missing id, bullet or section is an error naming it. `Added after Discussion` entries are appended. Correct.

## Findings

1. **MEDIUM, design.** Location: `internal/boardengine/raddlefile.go:81-121` (`FileRaddle`), read with `internal/boardengine/board.go:176-181`.
   - `boardCriticalSection` saves `board.json` at line 176 and runs `afterSave` at line 180.
   - The store directory, with `store.json` as the done marker, is written only in `afterSave`.
   - A crash, or an error from `writeRaddleStore`, between the two leaves the filed entries on the board with no marker.
   - The retry reads `StoreDone` as false and runs `verify`. `RecheckBoard` then reports every filed slug as `slug-taken`, so the proposal is sent back.
   - Each such send-back spends a return, so about two retries end in a halt.
   - The filed entries stay on the board and are never recorded.
   - D9 and the acceptance criterion require "every interrupted step finishes and repeats none".
   - Evidence: `raddlefile.go:90-92` (marker check), `raddlefile.go:118-121` (store written in `afterSave`), `recheck.go:38-41` (`takenSlug`).

2. **LOW, design.** Location: `internal/loomcli/raddleverbs.go:111-129`.
   - `raddle-propose` records `ProposedSession` before it posts the waiting notice.
   - If `notify` fails, the verb returns an error but `ProposedSession` is already set.
   - `raddle-approve` checks only `ProposedSession`, so it passes in a session that never sent its notice.
   - D7's Bound says the notice goes out in every session that approves.
   - Fix: set `ProposedSession` after the notice, or make approve require `NoticeSession >= Session`.
   - Evidence: `raddleverbs.go:115-121` and `raddleverbs.go:153`.

3. **LOW, design.** Location: `internal/loomshed/recordwrite.go` (`Call`, the `supportLog` variable).
   - `supportLog` defaults to `boardSupportLog()` and becomes `StoredRecordSupportLog()` only inside the `recordMissing` branch.
   - After a crash that wrote the stored record but not the log, the resume sees `recordMissing` false and `logMissing` true.
   - It then writes the board-rendered support log beside a stored record.
   - Evidence: the diff hunk of `recordwrite.go`, lines 136-168.

## Tool calls

1. `grep` of `internal/boardengine/board.go` and `internal/fsx/fsx.go`: needed the `afterSave` ordering against the save, and to know whether `AtomicWrite` creates nested directories. The manifest had only the diff hunks.
2. `grep` of `internal/state/state.go` for `WriteJSON`: needed to know whether it keeps undeclared keys. `shedrun` does not show that.
