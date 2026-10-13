---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/16-start-resume-go-driver.md:60
    summary: Card 16 makes resume a second reader of the recorded seed driver, but says the PATTERN background already names start and resume when it names one bootstrap verb, so no card amends PATTERN-driver-choice-single-site or its PATTERN.md line.
    fix: In card 16's Intent, replace the sentence "The background says the recorded driver is read in loom's bootstrap verbs `start` and `resume`." with a prescribed edit. `pattern/PATTERN-driver-choice-single-site.md`'s opening sentence says loom's recorded seed driver is read in its bootstrap verbs `start` and `resume` and in `lyx loom drive`'s own package, each branch selecting only the driving surface, as the decision record's first Constraint states. Add `PATTERN.md` to card 16's `Edit:` list. Add an Intent bullet that rewords its `PATTERN-driver-choice-single-site` line from "once per recipe, in its bootstrap verb" to "in its bootstrap verbs (loom's `start` and `resume`)", keeping the rest of the line. A validate-plan --overlay run with `PATTERN.md` added reports no new finding.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-drive-loop.md:63
    summary: At a friction failed done, card 11's notice names the stop report, but card 12's done writes no report, so the notice points at a file that does not exist, and D8's send line and its no-orch-strand rule are skipped.
    fix: "In card 11's Intent, say that for a hand-back describing a `friction: failed` done, `handBackRun` writes the report at `ReportPath`, sends the notice through `Notify` under the `OrchRecorded` rule and records the outcome line, but calls no `commitAndPark`. In card 12's `done` bullet, after `CommitRecords`, a `friction: failed` done builds a `driveReport` (run-id, position, reason, the friction directory), takes `NewReportPath` and calls `handBackRun` with it, so the notice names a report on disk. Add a `TestDriveHandBack` row: a friction-failed done writes the report with its send line and writes no park marker. Change card 12's `TestGoDriver_Settle` done row to assert that report on disk and no park marker. Text-only change, no target edits."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/20-batten-binary-restep.md:37
    summary: Card 20 writes the re-step marker before calling Spawn, but says a refused spawn is retried at the next due check, which that marker then blocks, since it already names the halt's stop key and build.
    fix: "In card 20's Intent, change the order so `binaryRestep` calls `Spawn` first and writes the marker (stop key and build label) only after `Spawn` returns nil. A refused spawn (`ErrChildNotParked`, `ErrChildDriverBusy`) writes no marker and is retried at the next due check. A batten crash between a spawn and the marker write is bounded by start's driver-lock check, which spawns no second driver. Add a `TestInnerRun_BinaryChangeRestepsADriverlessGoChild` row: a `Spawn` refusing with `ErrChildNotParked` writes no marker, and the next due check spawns again and then writes the marker. Text-only change, no target edits."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/12-drive-loop.md:56
    summary: Card 12 rechecks before every action but done, so a busy stop, whose run lock another process holds, usually exits through the recheck's running-exit, and D3's busy row never logs the lock holder.
    fix: "In card 12's step 4, change \"Before carrying out any action but `done`\" to \"Before carrying out any action but `done` and the busy exit\", so the busy row writes its `busy-exit` record and logs the holder as D3 says. The existing `busy writes busy-exit` row then holds with the run lock held by another process; say so in that row. Text-only change."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/02-loom-config-keys.md:51
    summary: The negative tackler-knob refusal that ValidateLocalValues adds to PrepareRunSettings has no test that reaches it; TestPrepareRunSettings gains driver_kind rows only, and TestLoadConfig_Refuses runs LoadConfig, not ValidateLocalValues.
    fix: "In card 2's Tests list, extend the `TestPrepareRunSettings` bullet with a `loom.tackler_max_spawns: -1` row expecting a refusal naming module `loom`, the key `tackler_max_spawns` and `wantInvalid`. Text-only change, no target edits."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/15-drive-verb.md:47
    summary: Card 15 wires the verify-failure record through the closures arbiterDepsFor builds, but that closure, PublishFailureTarget, returns only the target, and card 12's RecordMatchesHalt needs the record's row, head, target and write time.
    fix: "In card 15's `escalation and verify reads` bullet, say that `VerifyRecord` reads `verifytree.ReadPublishFailure` over `verifytree.NewPaths(location.WorktreePath(), verifytree.Dir(location.AnchorPath()))` and maps `Row`, `Head`, `Target` and `WrittenAt` onto `battenshed.ChildPublishFailure`, as battencli's `readChildPublishFailure` does. Say that `EscalationPending` wraps `arbiterDeps.PendingEscalation`'s bouncer read and drops its brief path. Text-only change."
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/13-drive-agents.md:36
    summary: Card 13 writes a spawn settle record before calling Tackler or Arbiter, so a spent budget, an off arbiter, a reused verdict or an attached live attempt each leave a spawn record for an agent that never started.
    fix: "In card 13's `tackle` step 1 and `arbitrate` step 1, move the `spawn` settle record to after the call returns. Write it only when the call started a fresh attempt, that is, when the returned attempt number is not among the attempts `OpenAttempts` or `ListTacklerAttempts` listed before the call. Write none for a refusal, a reused verdict or an attached attempt. Change the Budgets rows to assert no `spawn` record, and the restart rows to assert none for the reused attempt. Text-only change, no target edits."
---

### [BLOCKING] Resume becomes a second seed-driver reader, but no card amends the PATTERN that bars it

**Location:** `_lyx/plan/16-start-resume-go-driver.md:60`

**Issue:** Card 16 has `resumeFromRunState` read the seed through `shedrun.ReadSeed` and spawn `lyx loom drive` when it records `shedrun.DriverGo`.
That makes `resume` a second reader of the recorded driver, beside `start`'s `resolveSeedDriver`.
`pattern/PATTERN-driver-choice-single-site.md` opens: "A *recorded* seed driver value is read in exactly one place per recipe, that recipe's own bootstrap verb", and `PATTERN.md:54` says "Reading a seed driver: once per recipe, in its bootstrap verb".
The decision record's first Constraint widens this to "loom's bootstrap verbs (start, resume) and `lyx loom drive`'s own package".
Card 16 lists the background file under `Edit:`, but its Intent only says "The background says the recorded driver is read in loom's bootstrap verbs `start` and `resume`", which the file does not say, and prescribes no edit.
Cards 17 and 18 edit the same file, but only its permitted-consumer list.
No card targets `PATTERN.md`.
As planned, the code would contradict a listed PATTERN invariant that nothing amends, and CLAUDE.md requires the PATTERN change in the same commit.
The tripwire still passes, since it admits all of `internal/loomcli`, so no test catches the stale text.

### [MEDIUM] A friction-failed done names a stop report that is never written

**Location:** `_lyx/plan/12-drive-loop.md:63`

**Issue:** Card 11's `driveNotices` sends "a line naming the run-id and the stop report at a `friction: failed` done" under both rules (`11-drive-handback.md:57`).
Card 12's `done` action "commits the records through `CommitRecords`, sends the `friction: failed` notice when the envelope reports it, and exits", and its test row "`done` commits and exits with no park".
No report path is taken at done, since `NewReportPath` and the report write live only in `handBack` and `handBackRun`.
So the notice names a report that does not exist.
D8 also says "every send or failed send is a line in the stop report", and "With no orch strand recorded, the notice is logged and not queued, and the stop report alone carries it".
The `OrchRecorded` check sits inside `handBackRun`, which done never calls, so the done notice skips both rules.

### [MEDIUM] Batten's binary re-step marker blocks the retry the card promises

**Location:** `_lyx/plan/20-batten-binary-restep.md:37`

**Issue:** `binaryRestep` re-steps only when "the re-step marker ... does not already name this halt's stop key and this build".
It "then writes the marker (stop key and build label) before calling `Spawn`".
Line 39 says "A refused spawn (`ErrChildNotParked`, `ErrChildDriverBusy`) ... is logged and retried at the next due check".
After a refused spawn, the marker already names this stop key and build, so the next due check fails that condition and never retries.
The halt's one re-step for the deployed build is spent without a driver ever starting.
No row in `TestInnerRun_BinaryChangeRestepsADriverlessGoChild` covers a refused spawn, so the contradiction would ship either way.

### [LOW] The recheck preempts D3's busy row

**Location:** `_lyx/plan/12-drive-loop.md:56`

**Issue:** Step 4 runs `recheck` "Before carrying out any action but `done`", and `recheck` exits with a `running-exit` record on "a run lock another process holds".
A `busy` stop is the loop's refusal because another process holds the run lock (`Shed.Step` returns `ErrShedBusy` when `TryAcquireWriteLock` fails, `internal/shedengine/run.go:500-506`), so the recheck usually sees that lock still held.
The busy action at line 64, which "writes a `busy-exit` record, logs the holder named in the envelope's error", is then reached only when the holder let go between the step and the recheck.
D3's busy row asks for that log.
The test row "`busy` writes `busy-exit`" passes only if the fake reports the run lock free.

### [LOW] ValidateLocalValues' negative-knob refusal is reached by no test

**Location:** `_lyx/plan/02-loom-config-keys.md:51`

**Issue:** `ValidateLocalValues` applies "the `driver_kind` and negative-tackler-knob checks", and `PrepareRunSettings` calls it through the new `CheckValues` hook.
`TestPrepareRunSettings` gains only `loom.driver_kind: fast` and `loom.driver_kind: go` rows.
`TestLoadConfig_Refuses`' `tackler_timeout_min: -1` and `tackler_max_spawns: -1` rows go through `LoadConfig`, and the card does not say the two functions share one check.
So the run-settings refusal of a negative knob, which routes Seed-Child to Stuck, has no named test that reaches it.
`configengine.ValidateLocal` checks only that keys are open to the local layer, not their values, so no upstream check rejects the input first.

### [LOW] The verify-record wiring points at a closure that carries only the target

**Location:** `_lyx/plan/15-drive-verb.md:47`

**Issue:** Card 15 wires "`EscalationPending` and the verify-failure record through the same closures `arbiterDepsFor` builds".
`arbiterDepsFor` builds `PublishFailureTarget func() (string, error)` (`internal/loomcli/arbiter.go:561-564`), which returns only the record's target.
Card 12's `facts` calls `battenshed.RecordMatchesHalt`, which reads the record's `Target`, `WrittenAt`, `Head` and `Row` (`internal/battenshed/publishretry.go:116-145`).
`arbiterDeps.PendingEscalation` returns `(string, bool, error)`, not card 12's `(bool, error)`.
An implementer following the wiring text either has no record to match or writes a new reader the card does not name.

### [LOW] Spawn settle records are written when no agent starts

**Location:** `_lyx/plan/13-drive-agents.md:36`

**Issue:** `tackle` step 1 "writes a `spawn` settle record with settler `tackler`" before it calls `Tackler`, and `arbitrate` step 1 does the same before `Arbiter` (line 48).
`Tackler` can return `errTacklerUnavailable`, a verdict already on disk, or an attached live attempt (card 7), and `Arbiter` can refuse as off, ineligible or budget-spent.
In each of those cases no agent starts, but a `spawn` record is written.
D9 makes each settle record one Go action that happened, for `report-timing` to map onto its own categories.
The Budgets rows ("each hand back with no agent started") would then still find a `spawn` record.

## Rejected

- The goto-moved run being re-stepped rather than handed back as a `paused` stop: D12 says a moved run is acted on "instead of parking", the acceptance criteria name an orch goto during a tackler wait handled as a new stop, and card 13's row re-steps it, so the plan's reading is licensed.
- A race between batten's publish retry and an unwatched Go driver when batten runs without a recorded orch strand (no watched marker): card 12's recheck after a batten goto sees a moved run, so only one retry takes effect, and D4 defines watched by the marker.
- The tamper snapshot missing untracked files: D7 names "the existing dirty-paths read" for it, and card 7 states the bound.
