---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-driver-loop-to-sheddriver.md:56
    summary: Card 6 moves drive.go whole into sheddriver, but its resumeOpenAttempt reads an open arbiter attempt through loomengine.ReadArbiterAttempt, and no seam card 6 defines carries that read, so the moved file either keeps a loomengine import D1 forbids or loses the arbiter-attempt restart path.
    fix: In card 6's Intent, under the Arbiter *ArbiterSeam bullet, add a seam AttemptEnvelope func(attemptDir string) (envelopePath string, err error) that resumeOpenAttempt calls for the arbiter directory OpenAttempts returns when the tackler attempt yielded no envelope path; with Arbiter nil an open arbiter directory is ignored and the driver steps. State that loom's goDriverDepsFor fills it from loomengine.ReadArbiterAttempt's EnvelopePath. Add a TestGoDriver_Settle row to card 6's test list - a halted run whose only open attempt is an arbiter attempt rebuilds its stop from that attempt's envelope through the seam and calls the arbiter before any step - so the restart path stays pinned after the move.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-driver-loop-to-sheddriver.md:65
    summary: Card 6 declares TacklerDeps.Snapshot optional with nil recording an empty snapshot, and card 13 wires no Snapshot, but the tackler call that dereferences it (settleFresh in the tackler.go card 4 moves) is targeted by neither card 6 nor card 13, so batten's first tackler spawn calls a nil func.
    fix: Move the nil-safety into card 4, which already moves internal/loomcli/tackler.go and tackler_test.go - add to card 4's Intent, in the recipe-seams list, that TacklerDeps.Snapshot is optional and a nil Snapshot records an empty WorktreeSnapshot on the attempt, and add a TestTacklerCall_SpawnPath row in which the fixture's Snapshot is nil and the spawn succeeds with an empty recorded snapshot. Then reword card 6's sentence at line 65 to point at that behaviour (TacklerDeps.Snapshot is optional since card 4) instead of prescribing it.
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/plan/13-batten-drives-its-run.md:10
    summary: Card 13 adds battenDriveRefusal, a new refusal reachable through lyx batten run, in a module refusal-spec.md binds (its shed section carries the batten run rows), but lands no refusal-spec row for it, breaking PATTERN-refusal-way-forward.
    fix: Add `contracts/specs/refusal-spec.md` to card 13's Edit list (after `pattern/PATTERN-shed-verb-set.md`), and in the Intent's run-verb bullets state that the shed section gains a row titled batten run, a batten driver already drives the run - a second `lyx batten run <slug>` finds the driver lock held by a live batten driver, class transient, way forward `lyx batten status <slug>` shows the run and the live batten driver settles it. Name in the same bullet the input that reaches it (a second run while the first holds shedrun.DriverLock for the slug) and the test that reaches it - a TestRunCmd_HeldRunLockRefusesNamingTheLockPath row that holds the driver lock instead of the run lock and asserts the envelope carries battenDriveRefusal.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-module-overrides.md:80
    summary: The acceptance criteria require driver_kind in batten.yaml to be refused, but card 8 refuses it only in the local layer through configengine.ValidateLocal, while a hub base-layer batten.yaml holding driver_kind loads silently since the base layer keeps keys the template lacks.
    fix: In card 8's Validation bullets, state that battenshed.LoadConfig refuses a resolved document holding a driver_kind key, naming `batten config key "driver_kind"`, the reason (batten's own driver is always go) and the way forward (remove the key from batten.yaml), and that the refusal-spec row card 8 already edits covers it. Add a row to card 8's battenshed TestLoadConfig list - a base-layer batten.yaml holding driver_kind go is refused naming the key - beside the existing local-layer row.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/04-tackler-call-to-sheddriver.md:74
    summary: Card 4's Intent changes arbiterFake.Start to return a sheddriver.AgentRun, but its Edit bullet carries no re-sign arrow.
    fix: "Replace card 4's Edit bullet `internal/loomcli#arbiterFake.Start` with `internal/loomcli#arbiterFake.Start` -> `func (f *arbiterFake) Start(spec shuttleengine.Spec) (sheddriver.AgentRun, error)`."
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/07-shed-config-module.md:17
    summary: Card 7 plans TestResolveDriverKind and TestResolveTackler as one test per resolver, while internal/loomengine#TestResolveTackler, the test the two resolvers replace, pins both resolvers in one table.
    fix: Drop the Create bullet `plan:internal/sheddriver#TestResolveDriverKind` from card 7 and move its three rows (override set wins, empty override takes shed.yaml's, both empty takes llm) into TestResolveTackler's table, renaming nothing else. Lower the sheddriver budget figures by one - card 7's row rises from 15 to 18, card 10's from 18 to 19, and the overview's test-budget-rows decision reads 3, 9, 15, 18, 19.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/11-batten-loop-arming.md:17
    summary: Card 11 opens lyx batten step --until-stop with the idle watchdog disarmed, saying what it skips but not what bounds a step that wedges, as the attack-surface rule requires.
    fix: "Extend card 11's IdleTimeout bullet with the bound: a wedged batten step is never killed by the loop, as today's in-process generic run body kills none either, and it is bounded by the operator's `lyx batten pause <slug>` at the next producer boundary or by ending the detached loop job; the interrupted rule then applies to the killed step under Run-Shed's reinvoke policy and the repair cap."
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/13-batten-drives-its-run.md:77
    summary: Card 13 says the four TestRunCmd tests pin battenRunEnvelope's done, blocked and paused envelopes, but every existing row of those tests either completes to done or refuses, so no row ends blocked or paused.
    fix: In card 13's Tests list, add a TestRunCmd_ResumeDispositions row whose seeded status carries a pause_after naming the first row, so the in-process step stops paused, the driver hands back with no tackler, and the envelope's outcome is paused with ok true. Reword the sentence at line 77 to say the tests pin the done and paused envelopes, abandonedSession present and absent, and the busy message, dropping blocked, since a blocked stop reaches the Tackler closure that loads real config and a runner.
---

### [BLOCKING] Card 6 leaves the arbiter-attempt restart read on loomengine

**Location:** `_lyx/plan/06-driver-loop-to-sheddriver.md:56`

**Issue:** `internal/loomcli/drive.go`'s `resumeOpenAttempt` (lines 571-620) rebuilds a halted run's stop from the open attempt's saved envelope.
When the tackler attempt yields no envelope path, it reads the arbiter attempt with `loomengine.ReadArbiterAttempt(arbiterDir)` (drive.go:602).
Card 6 moves `drive.go` whole into `internal/sheddriver` and lists the edits the boundary forces "and no others".
Its `ArbiterSeam` holds only `EscalationPending` and `Judge`, and `OpenAttempts` still returns a bare arbiter directory.
`ArbiterAttempt` stays loom's type (D1 keeps the arbiter in loom), so nothing in the card lets the moved code read it without importing `loomengine`.
D1 and card 10's seam test forbid that import, and dropping the read loses the restart scenario the record keeps ("a driver restarted over a halted run reuses an unclosed tackler attempt" and its arbiter counterpart, pinned today by the `openArbiter` row at drive_test.go:350).

### [BLOCKING] TacklerDeps.Snapshot is made optional in a file no card targets

**Location:** `_lyx/plan/06-driver-loop-to-sheddriver.md:65`

**Issue:** `tacklerCall.settleFresh` calls `c.deps.Snapshot()` unconditionally (`internal/loomcli/tackler.go:184`).
Card 4 moves that file to `internal/sheddriver/tackler.go` and declares no handle for `settleFresh`.
Card 6 then says "`TacklerDeps.Snapshot` is likewise optional, nil recording an empty snapshot", but card 6's targets are the drive files, `drivewiring.go` and the budget file; the tackler call is not among them, and the shared decision bars later cards from naming card 4's moved file.
Card 13 wires "no `Snapshot`" (D4).
As planned, the first batten tackler spawn dereferences a nil func, and card 6's "no tamper seam" row does not catch it, because that row drives a fake `Tackler` closure, never the real `TacklerCall`.

### [BLOCKING] battenDriveRefusal lands without its refusal-spec row

**Location:** `_lyx/plan/13-batten-drives-its-run.md:10`

**Issue:** `PATTERN-refusal-way-forward` binds every module with a section in `contracts/specs/refusal-spec.md`.
The shed section already carries batten run rows ("Seed-Child: run settings invalid", "batten run: typed --child-driver over a seed recording none").
A new refusal there lands with its row in the same commit, and a test reaches it.
Card 13 adds `battenDriveRefusal`, the refusal of a second `lyx batten run` while the first holds the driver lock (an acceptance criterion), but card 13 does not edit `refusal-spec.md`.
`TestBattenDriverDeps` asserts only that `battenDriveLock` names the constant, which does not reach the refusal through the verb.
Loom's counterpart row, "drive: a Go driver already drives the run", shows the shape expected.
The overlay with `contracts/specs/refusal-spec.md` added to card 13's Edit list passes `validate-plan --overlay` alone and together with F5 and F6.

### [MEDIUM] batten.yaml driver_kind is refused only in the local layer

**Location:** `_lyx/plan/08-module-overrides.md:80`

**Issue:** The record's acceptance criteria say "`driver_kind` in `batten.yaml` is refused".
Card 8 relies on `configengine.ValidateLocal`, which runs only for the local layer (`internal/configengine/layers.go:247-251`).
The base layer is read without any check against the template (reconcile preserves keys the template lacks, `internal/configengine/set.go:38`).
`battenshed.Config` has no `DriverKind` field, so a hub `batten.yaml` holding `driver_kind: go` loads silently and is ignored.
The lens review noted the base-layer gap and fixed only the test's layer, so the criterion is met for the local layer alone.

### [LOW] arbiterFake.Start re-signs without the arrow

**Location:** `_lyx/plan/04-tackler-call-to-sheddriver.md:74`

**Issue:** Card 4's Intent says "loom's `arbiterFake` stays in `internal/loomcli/arbiter_test.go`, its `Start` returning a `sheddriver.AgentRun`".
Today it returns `arbiterRun` (`arbiter_test.go:43`), so the method's results change, and the Edit bullet needs the re-sign arrow.
The arrowed bullet passes `validate-plan --overlay`.

### [LOW] Card 7 splits the resolver test per symbol

**Location:** `_lyx/plan/07-shed-config-module.md:17`

**Issue:** `internal/loomengine#TestResolveTackler` (`internal/loomengine/driver_test.go:115-177`) pins both `ResolveTackler` and `ResolveDriverKind` in one test.
Card 7 replaces the two resolvers with `sheddriver`'s and plans one test each: `TestResolveDriverKind`, with three rows, and `TestResolveTackler`.
Two pure resolvers over the same `Config` and `Override` inputs are one behaviour's table, and `PATTERN-test-economy` rules out the one-test-per-symbol shape.
Dropping the Create bullet passes `validate-plan --overlay`.

### [LOW] Card 11 names no bound for a wedged step

**Location:** `_lyx/plan/11-batten-loop-arming.md:17`

**Issue:** Card 11 makes `lyx batten step <slug> --until-stop` a live entry and leaves `IdleTimeout` zero and `Activity` nil.
It gives the reason, Run-Shed's idle wait, but not what bounds a batten step that wedges, which the loop's watchdog bounds for loom.
Nothing is newly unbounded, since today's generic run body has no watchdog, but the attack-surface rule asks the Intent to state the bound.

### [LOW] The run tests do not pin the blocked and paused envelopes

**Location:** `_lyx/plan/13-batten-drives-its-run.md:77`

**Issue:** Every `TestRunCmd_ResumeDispositions` row completes the run to done (`internal/battencli/run_test.go:131-190`).
`TestRunCmd_StateDoneRefusesNamingTheRunDir` and `TestRunCmd_HeldRunLockRefusesNamingTheLockPath` refuse, and `TestRunCmd_AbandonedSessionKey` ends done.
So no row reaches `battenRunEnvelope`'s blocked or paused outcome, and the card's claim that they are pinned is wrong.
A paused stop hands back by the rule table without a tackler, so a paused row can be reached over the bare temp hub.
A blocked stop reaches the `Tackler` closure.

## Rejected

- Card 6's Edit group lists `internal/loomcli/drivewiring.go` as a file, although its Intent names `driveCmd`, `goDriverDepsFor`, `writeDrivePausedNote` (which re-signs to take a `sheddriver.Stop`) and `newestAttemptsFor`.
  The member-glyph form cannot cover `driveCmd`.
  Its glyph `internal/loomcli#loomCLI.driveCmd` resolves into `internal/loomcli/drive.go` at plan time, and card 6 moves that file, so `validate-plan --overlay` refuses it with `redundant-file-target`.
  Without `driveCmd` the member list leaves that change untargeted, so the whole-file target is the admissible shape and is not a finding.
- Card order: cards 3, 4, 6 and 13 reach moved names through earlier cards' `plan:` handles or unmoved glyphs.
  Card 3 edits `goDriverDeps` and `goDriver.reportFor` before card 6 moves them, card 13's `driverReportPath` stays in `internal/loomcli/driverreport.go`, and `driveEnvelopePath` stays in loomcli after card 5.
  No card depends on a later one.
- Test-budget rows: current counts are `internal/battencli` 79/79, `internal/battenshed` 111/114 and `internal/loomcli` 235/237, and the moved test counts are 3 for card 2, 6 for card 4 and 6 for card 6.
  They match the Shared Decisions' figures: sheddriver 3, 9, 15, 19, 20; battencli 80, 81, 81; battenshed 113 of 114.
- Fidelity: every Decision D1-D9 and every Constraint is carried by a card, apart from F4's gap.
  That includes D5's comment-refresh step, which card 9 builds as step 2 of `migrateShedOverrides` with idempotence and the hand-edited-comment row.
  Card 10's shortening of three PATTERN entries is forced by `HeadroomBytes` (15561 of 15564 bytes used), not added scope.
- Verify: `## verify:` runs `./...` in the template's order, covering `internal/sheddriver`, `internal/battencli` and `cmd/lyx`, with the integration tier run and the tmux and llm tiers compiled.
