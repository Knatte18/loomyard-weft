```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff through the code-correctness lens and report each function or branch, the case I ran, and whether it was correct.

## Lens list: function or branch, case run, verdict

- `Run` (`run.go:140`):
  - Held run lock → `ErrRunBusy` with way forward: correct.
  - Corrupt state file → Stuck with a placeholder `State`: correct.
  - Stale `outcome.yaml` and `summary.md` removed on entry: correct.
- `beginBatch` (`run.go:274`):
  - First begin records partition, `StartSHA` and records head: correct.
  - Re-begin of a non-terminal batch whose card set now differs → refusal naming the reset way forward: correct.
  - Its doc comment says it clears the stop record and the code does not: finding 5.
- `runBatch` (`run.go:393`):
  - Failed attempt → outcome recorded in `endAttempt`, then strand stopped in `settleAttempt`, then next rung: correct, D6 ordering holds.
  - One lost attempt on a rung → same-rung retry; second loss → climb (`lostOnRung` includes the current attempt): correct.
  - `StartGated` error → lost with an empty guid, `stopAttempt("")` does nothing: correct.
  - State write fails after a successful stop → error return, next call probes and counts a loss: correct.
  - Failed stop with a `NEEDS_PARENT` outcome → stop kind overwritten: finding 1.
- `settleAttempt` (`run.go:538`):
  - Done → no stop, correct.
  - `endErr` set → strand stopped, `InFlight` left set and unsaved. This is safe only because the next call re-stops idempotently (`StopStrand` falls back to `removeStrandIfPresent`, `stop.go:72-78`): correct.
  - Stop failure → `stopErr` with the attempt still marked in flight, so a re-entry stops it again: correct.
- `nextAttemptRung` and `lostOnRung`: first loss retries, second climbs, a crash does not reset the count, and the count restarts from the latest `Reentry` attempt: correct.
- `endAttempt` and `checkRecordable` (`record.go`):
  - Dirty tree → failed with `Uncommitted`: correct.
  - Abbreviated `head_sha` resolved through `CommitsNamedBy`: correct.
  - Journal appended before the state save, keyed by heading: correct.
  - A full 40-digit upper-case `head_sha` is not lower-cased and fails the first-parent walk as "not HEAD": finding 8.
- `finishRun`, `nextFixerRung`, `awaitFixer`:
  - Verify fail → fixer → verify again; a lost fixer retries on the same rung; a re-entry fixer runs on the last rung and a second failure stops: correct.
  - A runner crash gets no fixer: correct.
  - A fixer stop-failure warning → Stuck with guid, state already recorded: correct.
  - A runner-crash stop records `Rung: len(st.Fixers)`, which is meaningless but unused by re-entry: not a defect.
- `resumeStrands`, `stopRecordedStrands`, `resumeAttempt`, `resumeFixers`, `probeUnrecordedFixer`:
  - Recorded-outcome strands stopped first; a re-stop is safe; a probe error → Stuck with nothing spawned; a report with no run behind it counts as lost: correct.
  - A found fixer is attached; an unrecorded fixer is probed and counted lost: correct.
  - `settleAttempt` in the resume path returns its stop failure as a cause with no `StopRecord`, so `judgeOpenBatch` judges from the last attempt and `NEEDS_PARENT` survives: correct.
- `judgeOpenBatch`: lost-retry and climb are correct. A `fault` outcome climbs a rung: finding 7.
- `reenter`:
  - A `NEEDS_PARENT` stop returns the asking rung and never takes the no-spawn path: correct.
  - A verify stop returns Verify: correct.
  - A batch stop with passing checks and no row gates → done with no spawn; with row gates or failing checks → one last-rung attempt: correct.
  - A card-set mismatch → refusal naming `reentryWayForward`, via `MapPartition`: correct.
- `Reset` (`reset.go:91`) and its helpers:
  - Held run lock → refusal: correct.
  - Absent state with marker pending → commit finished; no marker → refusal: correct.
  - Corrupt state → newest parsing version, reed live-strand refusal, manual-repair refusal when none parses: correct.
  - Probe of every recorded report and reply path before moving anything, with `--stop-strand` stopping through the handle: correct.
  - A failed commit leaves the marker and the re-run finishes it: correct.
  - Ordering of the foreign-path check against the strand probe: finding 2.
  - Run lock released before the move: finding 3.
- `PendingPartition` and `RecordBeginPartition` (`pending.go`): slug matching; done batches kept in recorded order; fresh partition over the remaining cards; none when all are done: correct.
- `Active`, `ActiveOrSetting`, `restorableVersion`, `implementerGuard` (`active.go`, `loomshed/weaver.go`):
  - A missing file counts as no begun batches, a corrupt file refuses, both begun refuses with both resets named: correct.
  - The repair is offered only when the newest parsing version's `LastEnd` equals the task HEAD and no Weaver strand is live: correct.
  - The advisory fallback to the setting: correct.
- `LoadConfig`, `ResolveLadder`, `weaverEntry`: empty ladder, bad spec, non-positive timeout and empty verify gate each refuse naming the key and `lym config weaver`; the gates key is forwarded: correct.
- `WeaverProducer.Call` (`shedadapters/weaver.go:66`): done, stuck, error and unknown-outcome mapping and the conflict claim are correct. No context reaches `Run`: finding 6.

## Findings

1. **MEDIUM, design** — `internal/weaverengine/run.go:510-512`. When the strand stop fails, `runBatch` records the stop as `stopKindBatch`, whatever the attempt's outcome.
   - Case: a `NEEDS_PARENT` attempt whose `StopStrand` fails. `stop(stopKindBatch, rung, …)` overwrites the kind, which `nextAttemptRung` had set to `needs_parent` only on the success path.
   - Re-entry then takes `reenter`'s batch branch. That branch can record the batch done with no spawn, which D7 forbids after `NEEDS_PARENT`. It also restarts on the last rung rather than the rung that asked.
   - A failed stop of any failed attempt on a non-last rung likewise skips the remaining rungs.
   - The resume path (`resume.go:311-332`) does this correctly, so the two paths disagree.
   - Evidence: `run.go:507-516`, `resume.go:360-406`.

2. **MEDIUM, design** — `internal/weaverengine/reset.go:128-134`. `refuseForeignPaths` runs before `settleRecordedStrands`, and `own` holds only paths recorded on failed attempts.
   - Case: a live in-flight attempt has uncommitted edits. They are foreign, so the reset refuses with "commit them or git checkout", which is wrong advice while an agent is still writing.
   - `--stop-strand` therefore cannot be reached for a typical live attempt. After a stop, the stopped attempt's dirty paths are still not in `own`, so the user must clean them by hand first.
   - Evidence: `reset.go:128-134`, `reset.go:244-256`, `weavercli/wiring.go:117-140`.

3. **LOW, design** — `internal/weaverengine/reset.go:96-103`. `Reset` probes the run lock and releases it at once. It holds neither the lock nor the state-mutation lease while it archives and clears, so a row stepped between the probe and `archiveAndClear` runs against a half-reset tree.

4. **LOW, design** — `internal/weaverengine/run.go:687-690`. `stopReason` cuts `cause` at 600 bytes, mid-rune and at the end of the text. `StrandStopError`'s `lym reed remove &lt;guid&gt;` way forward sits at the end of its message, so a long stop error cuts the way forward D6 requires.

5. **LOW, consistency** — `internal/weaverengine/run.go:272`. The `beginBatch` doc says it "clears the stop record"; the code never does. That is correct, because re-entry needs it to mark `Reentered`, but the comment is wrong.

6. **LOW, design** — `internal/shedadapters/weaver.go:71` and `weaverengine/run.go:140`. `Run` takes no context, so a cancelled shed call cannot stop the batch loop; it keeps starting strands until it ends. Only the verify's pause hold is honoured. I did not check how the driver ends a producer.

7. **LOW, design** — `internal/weaverengine/resume.go:323`. A `fault` outcome (Weaver's own check or write failed) falls into the climb branch of `nextAttemptRung`. A re-step after such a fault therefore spends a model rung on a fault that says nothing about the model.

8. **LOW, design** — `internal/weaverengine/record.go:214`. Only abbreviated hex heads are lower-cased, so a 40-digit upper-case `head_sha` fails the first-parent walk as "not HEAD".

## Tool calls

1. Grep of `StopStrand` in `internal/shuttleengine/stop.go`: whether a repeated stop of an already-stopped strand errors; my context lacked shuttle's code.
2. Grep for "stop strand" warnings and `Status*` constants in `internal/gatefixer`: whether `awaitFixer` matches the warning text `gatefixer` emits and which statuses exist.
3. Grep of `ResetCode` and `DirtyTracked` in `internal/weavercli/wiring.go`: how the reset's seams are wired; not in context.
4. Read `internal/runplan/pending.go`: the partition code behind D3 and D11; not in context.
