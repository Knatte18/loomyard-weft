```yaml
lens: code-correctness
outcome: reported
```

**Task: lens code-correctness. I ran each changed function against concrete inputs, including failure cases.**

**Per-function list**

| Function / branch | Case I ran | Result |
|---|---|---|
| `Record.Covers` | Done record; `root` with `PrimeRecorded`; beam in `OwnerStood` | Correct |
| `Watch.Covers` / `covers` | No record; undecodable file; board read error; unfinished record | Correct. A failed read returns false, logged once, and the board is not read without an unfinished record. |
| `ownerSubject` | Prime (`""`); beam with its own owner; beam with no owner under `root`; entry not on the board | Correct for these. **Wrong** for a beam adopted by name with no owner of its own (F1). |
| `probeOwner` | Owner not foreign; foreign with hold; foreign without hold; owner read fails | Correct. A failed owner read reads as not foreign, so the hold lifts. That matches today's fail-open behaviour. |
| `handOver` | Archive ok; archive fails then ok; `ArchiveOwner` nil | Correct order: pause, archive, exit. A failure keeps the dobby up and retries. Episode reset is missing (F2). |
| `pauseRunsForOwner` | Runs error; state error; done run; run already `OwnedElsewhere` | Correct and skips as intended. A run whose pause failed has `OwnedElsewhere=true`, so it is not re-paused. That is the "full pass each probe" bound. |
| `pullBolt` (dobby) | First probe; within 5 minutes; failing pull | Correct. `lastBoltPull` is set before the pull, so a failed pull waits a full interval. |
| `tick` hold gating | `adoptHolds` true | `drivePasses`, `startReady`, tidy, test-map and `landBeam` are skipped, and `owesAll` is kept. Correct. |
| `ArchiveOwnerTips` / `archiveOwnerPair` | Lock busy; origin unreachable; two pairs where one fails | Correct. Every pair is tried, failures are joined and wrap `ErrArchiveFailed`, and the lock is released by defer. |
| `archiveOwnerTip` | No dir; no origin remote; branch absent on origin; origin equals tip; origin ahead and holds tip; diverged; tag exists at the same sha | Correct. `ls-remote` is live, so a stale tracking ref is not an issue. A rerun reuses the tag and repushes it. |
| `createTagAt` / `pushArchiveTag` | Refactor from `createArchiveTag` / `tagAndPushArchiveTip` | Behaviour preserved. |
| `archiveBeforeOwnerRefusal` | Owner foreign and archive ok; archive fails; owner local; archive nil | Correct. It also runs for a non-owner refusal when the owner is foreign (F3). |
| `ownerArchiveBeforeRefusal` / `archiveOwnerTips` (beamcli) | Prime worktree (slug `""`); beam; a run whose worktree is gone | Correct. |
| `orchengine.ownerTick` | Foreign with and without hold; owner read error | Correct. The "no adopt covers" log also fires when the owner simply returns to this machine (cosmetic). |
| `orchcli` lazy accessor | Build failure | Fails open to no adopt, with one log per episode. That matches D4. |

**Findings**

1. **MEDIUM, design.** `internal/adoptrecord/watch.go:124-136` (`ownerSubject`, used by `covers` at line 118).
   - A beam named in the record as its own subject, with no `Owner` recorded on its entry, resolves to its parent subject (`root`) and not to itself.
   - Input: `lym adopt b` where `b` is a beam with no owner of its own under a foreign-owned `root`. The record lists `["b"]` and nothing has been written yet.
   - Trace: `ownerSubject` returns `root`, and `record.Covers("root")` is false because `root` is not listed. The beam's dobby and orch therefore take the foreign-owner path and archive or exit during the adopt.
   - D4 says a subject covers its own worktree, so this contradicts it.
   - Evidence: `watch.go:129` requires `entry.Owner != ""` before the slug counts as its own subject; the tests only use beams that already have an owner (`watch_test.go`, rows "AStoppedBeamAdoptStillCoversTheBeam" and "ARootAdoptLeavesABeamWithItsOwnOwner").
   - Fix: check the slug and each ancestor against `record.Covers` as the chain is walked, and stop at the first beam with its own owner. Add a row for a beam in the record with no owner.

2. **LOW, design.** `internal/dobbyengine/handover.go:403-410`. `archiveFailures.Recover` is never called, and `archiveFailing` is never cleared.
   - If a failed archive is followed by the owner returning to this machine (hold or no longer foreign), a later failed archive in a new episode logs nothing to the dobby log and nothing to the logger.
   - Fix: call `archiveFailures.Recover(...)` and clear `archiveFailing` in `probeOwner` when `!elsewhere || holds`.

3. **LOW, design.** `internal/orchcli/start.go:508` and `start.go:841-851`. The archive runs for any refusal whenever the owner is foreign, not only for the foreign-owner refusal.
   - If `beamUpRefusal` returns the anchor-move or hold refusal first while the owner is also foreign, the archive still runs.
   - Its failure text is appended to a refusal that is about something else.
   - Fix: run the archive only when the refusal is the owner refusal. Compare against the sentinel, or make `beamUpRefusal` report which refusal it returned.

**Tool calls**
- Read `internal/dobbyengine/owned.go`: `ownedElsewhere` semantics were not in context.
- Read `internal/dobbyengine/loop.go:500-710`: the full tick and loop bodies were not in context.
- Read `internal/fabricengine/remoteleftover.go:195-260`: `remoteHeadTip` and `isAncestorOrEqual` were not in context.
- `grep` for `Tick:` and `adoptHolds` / `OwnedElsewhere`: to find other schedulers that bypass the hold. I found none beyond `deps.Tick`, whose body I did not read (`internal/beamcli/dobbydeps.go:217`).
