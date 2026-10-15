# Card 2 — conflict-record

**Create:**
- `internal/websterengine#ConflictRecordRel`
- `internal/websterengine#ConflictRecordPath`
- `internal/websterengine#RoutedConflictPath`
- `internal/websterengine#WriteConflictRecord`
- `internal/websterengine#RemoveConflictRecord`
- `internal/websterengine#RemoveRoutedConflict`

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine#Geometry`
- `internal/websterengine#ScratchDir`
- `internal/websterengine_test#TestRun_RefusesBeforeSpawn`
- `internal/websterengine_test#TestRun_EntryHousekeeping`
- `internal/websterengine_test#TestRun_AutoRebaseline`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/webstercli#TestBeginBatchCmd_PlanConflict`
- `internal/webstercli#runVerbs`
- `internal/websterengine/doc.go`

**Intent:**
Decision D2: begin-batch leaves a Go-written, never-tracked conflict record that names the blocking findings, bounded to one Webster call.

A new file `internal/websterengine/conflictrecord.go` declares the two scratch files under the told `Geometry.ScratchDir` (`.lyx/webster/`, never `_lyx`, per `PATTERN-durable-vs-ephemeral-state`; the literals come from `lyxdirs` as `ScratchDir` builds them, per `PATTERN-lyxdirs-single-declarer`):
the conflict record `plan-conflict.md` and the last-routed conflict `plan-conflict-routed.md`.
`ConflictRecordPath` and `RoutedConflictPath` join each name onto `scratchDir`;
`ConflictRecordRel` returns the record's anchor-relative, forward-slashed path `.lyx/webster/plan-conflict.md`, built with `path.Join` from the same declarations, for the recipe text card 3 pins.
`WriteConflictRecord` writes the record as markdown: a heading naming the batch (`# Plan conflict at begin-batch NN`), a `cause: <cause>` line, a blank line, then each finding of `conflict.Findings` on its own `- <finding>` line.
Every caller holds a lock file in `scratchDir`, so it already exists and the write creates no directory.
`RemoveConflictRecord` and `RemoveRoutedConflict` remove their file and return nil when it is absent.
The `ScratchDir` accessor's comment and the `Geometry.ScratchDir` field comment, which list what the scratch dir holds, add the two conflict files.

`Run` calls `RemoveConflictRecord(deps.Geom.ScratchDir)` right after it acquires run.lock (after the `ErrRunBusy` return, before the plan parse and so before the auto-rebaseline), so a Run refused with `ErrRunBusy` never removes the live run's record and an auto-rebaseline refusal never sees a stale one;
a removal error is logged at Warn and changes nothing else.

`beginBatchCmd` calls `websterengine.RemoveConflictRecord(c.geom.ScratchDir)` as the verb's first step, before the batch-number `Atoi`, logging a failure at Warn, so a begin-batch that refuses for any other reason (an invalid batch number, a plan parse failure, quarry-unavailable, a fingerprint mismatch, any later refusal) leaves no record.
On a `*websterengine.PlanConflictError` whose `persistPlanFingerprintRebaseline` succeeded, it calls `WriteConflictRecord` before the mutate lock is released;
a write error is logged at Warn and the refusal's message and `plan_conflict` class are printed unchanged.
A failed persist already prints the combined refusal with no class, so it writes no record, and the run halts as today.
After a successful `SaveState`, under the lock, it calls `RemoveRoutedConflict`, logging a failure at Warn;
the record itself is already gone from the first step.
So a record exists only when the latest begin-batch of this Webster call refused with `plan_conflict`.
`internal/websterengine/doc.go` gains a section on the conflict record: its two files, who writes and removes each, and that it is never tracked, so no sync carries it.

Tests:
`runVerbs`, whose bare `websterCLI` carries no geometry, sets `geom.ScratchDir` to a temp dir, so the new first-step removal never resolves against the test's working directory.
`TestBeginBatchCmd_PlanConflict` seeds a record through `WriteConflictRecord` before every row and gains, per row, the record's expected state afterwards.
Its card-1 row asserts the record holds the batch heading, `cause: plan_defect` and both blocking findings' lines.
New rows:
- the record path pre-seeded as a non-empty directory, so both the first-step removal and the write fail, still refuses with the same message and `plan_conflict` class;
- `begin-batch x` (invalid number), a plan whose `00-overview.md` was removed (parse failure), a card file edited after the state was seeded (fingerprint mismatch, refused by `PlanEditError`), and `geom.WorktreeRoot` pointed at a missing directory (quarry-unavailable) each refuse with no `plan_conflict` and leave no record;
- a successful `begin-batch 1` over a clean plan, with a routed file seeded at `RoutedConflictPath`, exits 0 and leaves neither file;
- a successful `begin-batch 1` whose `RoutedConflictPath` is a non-empty directory still exits 0 with the directory in place.
`TestRun_RefusesBeforeSpawn`'s run.lock row seeds a record and its `check` asserts the record is still there after `ErrRunBusy`.
`TestRun_EntryHousekeeping` gains a row seeding a record that asserts Run removed it before stopping, and a row whose record path is a non-empty directory asserting the scripted starter was still reached (`stop before spawn`) with the directory left in place, so a removal failure that refused would fail the row.
`TestRun_AutoRebaseline`'s refusal subtest seeds a record before the refused run and asserts it is gone afterwards, proving the removal precedes the auto-rebaseline.
`internal/websterengine` is at its test budget, so every engine test is a row.

**ImpactSummary:** Adds two never-tracked scratch files and three call sites (Run entry, begin-batch first step and its two exits); refusal messages, classes and the Run outcome are unchanged, and no other verb reads the files yet.
