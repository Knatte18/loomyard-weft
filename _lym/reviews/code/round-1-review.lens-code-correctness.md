```yaml
lens: code-correctness
outcome: reported
```

**Task restated:** I checked the code-correctness of the diff by running concrete cases through each changed branch.

## Branch list (case, verdict)

- **`FileRaddle` family branch and `convertRunEntry`** (entry with a stored record; entry with none; failure after the record write): correct.
  - A held record is kept in `formerRecords` and restored when the write fails and `saved` is false.
  - With no former record, the path is removed, and it is only the entry's own new record, so that is right.
  - A refusal raised before `convertRunEntry` leaves `formerRecords` empty.
- **`arrasProjectOf`** (empty former body; non-empty former body): correct. The body is the prefix, the part list and `.\n`, plus `\n` and the former body only when one exists.
- **`endsAsFamilyProject`** (empty part list; part out of dependency order; undiscussed part; another producer; changed title or `depends_on`): correct.
  - `scopeChanged` compares `Order`, `Body`, `Labels`, `Title`, `Kind` and `DependsOn`, so the derived shape is pinned. `Status` and `Discussed` are checked separately.
  - Parts are read from `after`, never from the entry's `order`.
  - An empty part list returns false.
- **`raddleSinkParts`, `repointedDependencies`** (waiter depending on `other` and `plan`; no sinks): correct. The slug is replaced in place and deduplicated. An empty sink list is unreachable because a filing always has parts.
- **`repointDependents`** (dependent held by a run; dependent the board refuses; done dependent): correct.
  - A held dependent is refused with `ErrRaddleRepointRefused` wrapping `RunLockedError`.
  - A dependent the board refuses is refused with `ErrRaddleRepointRefused` naming the entry.
  - A done dependent is skipped.
  - The refusal returns before the save, so the record file is restored and nothing is left filed.
- **`Call`: halt over a foreign project** (`filed` false, `resumed` false, entry kind project): correct. It sits before `verify`, as D6 requires, and files nothing.
- **`Call`: repair path** (store done; `AlreadyFiled` with the stage present): correct. The mark is written before the status clear, and `finish` skips `EndEntry` when `family` is true. In the `AlreadyFiled` case, `stageAndFile` runs in store-only mode, so there is no second board write.
- **`Call`: first filing** (`family`, not repaired): correct. `MarkFiled` runs only after `stageAndFile` returns with no error and no send-back reason.
- **`finish`** (`family` true, beam shape, notify done): correct. Only `EndEntry` is skipped.
- **`ClearProjectRunStatus`** (absent entry; task kind; project with a run status): correct.
- **`ClearMotifRun` with arras and the mark** (mark present; commit failure): correct. `removeRunRecords` removes the whole `RunDir`, so the mark goes with it. The restore writes it back when `filingMarked` was true.
- **`skipAfterFamilyFiling`, `raddleFilingMarked`** (mark read fails; `SelfRunID`): correct. A read failure writes nothing and returns the error. `ResolveRunID` on the prime resolves to the same slug `MarkFiled` uses. The base `SetBoardStatus` is non-nil (`wiring.go:220`).
- **Batten `SetBoardStatus` skip:** correct.

## Findings

**F1 — LOW, design.** `internal/loomshed/raddlefile.go:177-183, 209`. A repaired family filing can write a run status back onto the converted project.
- `Call` reads `entry` once, before `repairFamily` clears the status. In the `AlreadyFiled` case (store write lost), `stageAndFile` then tests the stale `entry.StatusProducer`.
- If that producer is non-empty and not `Raddle-File`, `ClaimStatus` runs `SetRunStatus` on the converted project after `ClearStatus` has already cleared it.
- The project is then left with a run status, against "releasable with no status".
- Reaching it needs a non-Raddle-File status on a project: the hook failed at the move into the row, and a stale status from another producer persisted.
- Fix: in `stageAndFile`, skip `ClaimStatus` when the call is `resumed`, or when `repaired` is true. A repaired filing writes no board entry.

**F2 — LOW, design.** `internal/loomshed/raddlefile.go:177-178`. `family` is `parseErr == nil && ...`, so `family` is false whenever the parse failed.
- If the store is done (`filed`) and the proposal file no longer parses, then `repaired` is false and `finish` would run with `family` false.
- `EndEntry` would then mark the converted project done.
- This needs a corrupt proposal alongside a done store, which is unlikely.
- Fix: when `filed || resumed` and `parseErr != nil`, fail at the parse before `finish`. Calling `p.fail("parse the proposal", parseErr)` right after the `StoreDone` read does it.

No other defects found in my lens.

## Tool calls
1. Read `internal/loomshed/raddlefile.go` lines 134-223: the diff hunk did not show where the entry halt sits relative to `verify`.
2. grep `scopeChanged` and `ResolveRunID`: to confirm which fields the lock compares and how the run id resolves.
3. grep `removeRunRecords` and the start of `FileRaddle`: to confirm the removal takes the mark and to see the early-return structure.
4. grep `SetBoardStatus` in `internal/loomcli` and `SetRunStatus` in `internal/boardengine/board.go`: to check for a nil base and the status-write semantics.
