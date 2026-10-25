---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/33-dobby-passes.md (SpawnPass) and _lyx/plan/28-batten-first-pass.md (StartFirstPass)
    summary: Cards 33 and 28 start every lyx batten run pass with the hub root as its working directory, where lyxcwd.Resolve refuses as not a git repository, so no detached pass can arm.
    fix: In card 33's Intent, change SpawnPass so it starts `<lyx> batten run <slug>` with the prime's anchor path (`location.AnchorPath()` of the prime, carried in dobbyengine.Geometry as the prime's anchor) as its working directory, not the hub root. In card 28's Intent, do the same for StartFirstPass's detached `<lyx> batten run <slug>`. Keep the hub root only as the dobby's own working directory (card 27's SpawnDetached, D1). State in both cards that the pass resolves through batten's own pre-run, which needs a worktree anchor, as the window run's `OpenWindowIn(..., c.location.AnchorPath(), ...)` and orchcli's spawn (`cmd.Dir = c.location.AnchorPath()`) do today.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:affected-set
    location: _lyx/plan/13-innerrun-short-check.md, _lyx/plan/28-batten-first-pass.md, _lyx/plan/04-dobby-records.md, _lyx/plan/35-dobby-ready-starts.md, _lyx/plan/36-dobby-board-tidy.md
    summary: The plan does not list the tests, callers and import allowlists that a card's change forces outside its own package, so cards 13, 28, 35 and 36 each fail their card gate or break the build.
    fix: Add these targets under each card's **Edit:** and a sentence on each to its Intent. Card 13 gets `internal/battenshed#battenshedAllowedImports`, which gains `internal/state`, plus `internal/battencli#wireForHub`, `internal/battencli#stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`, `internal/battencli#stepRunShedPausedChild_WaitsThenTearsDownOnceDone` and `internal/battencli#stepAwaitingChildDoneTeardown_ArchivesTheRunRecords`, each rewritten to drive passes and expect `Yield` where the in-call wait used to keep going. Card 28 gets `internal/battencli#battenCLI.registerSeededRun`, which loses its windowFlag check, plus `internal/battencli#TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`, `internal/battencli#TestArmSeed_RunAndStepSeedBeforeWire` and `internal/battencli#stepCommandRows_FollowTheTasksLocalSwitch`, which lose their windowFlag and openTerminalFlag uses. Card 4 gets a new **Create:** bullet, `plan:internal/dobbyengine#dobbyengineAllowedImports` -> `var dobbyengineAllowedImports []string`, holding only card 4's own imports. Cards 35 and 36 each get `plan:internal/dobbyengine#dobbyengineAllowedImports`, adding the imports each card brings in, boardengine on card 35, and shuttleengine, stencilstore and any other import on card 36. Any of cards 32 to 34 that adds an import edits it the same way. The plan gate's `--overlay` run accepts these targets.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/13-innerrun-short-check.md (Intent, childWait.begun)
    summary: A child state change between passes cannot end the Call as card 13 promises, because Call's observeState replaces the persisted episode before begun is read, and checkChild skips an unchanged file, so a running-to-done child can return Done with no state-change entry.
    fix: In card 13's Intent, state that Call takes begun from the memory record's persisted episode state as loaded by loadMemory, before observeState runs. State also that when status.State differs from that begun, Call ends at once with the state-change Stuck (`stateChangeText(begun, status)`) before any arm runs. Add a row to `TestInnerRun_MemoryOutlivesTheProcess` in which a pass sees the child running, the next pass finds it done, and that pass returns the `child running → done` Stuck before Done.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/25-batten-arbiter-wiring.md (Intent, loadedArbiterDeps, no goto guard)
    summary: Card 25 gives batten's arbiter a goto with no goto guard, but its Intent states neither what a goto back can re-run nor what bounds it, while batten's tackler may resume only the halted row.
    fix: "In card 25's Intent, name the bound: shedengine.CheckGotoTarget admits only rows at or before the current one. Then give batten's arbiter deps the tackler's bound, a GotoGuard in loadedArbiterDeps that refuses any target but the halted row, with a refusal naming `lyx batten goto <slug> --to <row>` for the parent. State that a back goto would otherwise re-run Seed-Child, which rewrites the task worktree's local config layer and seed, or Worktree-Create, over a live pair. Add a TestBattenDriverDeps row in which a goto to Seed-Child from Land is refused."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/04-dobby-records.md (ReadState, UpdateState), _lyx/plan/31-batten-pass-wiring.md (dobbyReading), _lyx/plan/32-dobby-loop.md (tick)
    summary: A dobby state record that does not decode wedges the beam with no way forward, since UpdateState fails on every tick and card 31 returns the read error into every pass's wiring.
    fix: In card 4's Intent, state that ReadState and UpdateState read an undecodable record as absent with one Warn, as card 13's loadMemory does, and that UpdateState then writes a fresh record. Card 32's start then reads that fresh record as a start, which owes every run a full pass. In card 31's Intent, state that dobbyReading reads an undecodable record as this process's pid and no sleeps, with one Warn, rather than returning the error. Add a row to card 4's TestState_RoundTripKeepsUnknownKeys for an undecodable file read as absent and rewritten. Change card 31's TestDobbyReading row so an undecodable record reads as no dobby.
  - id: F6
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/plan/17-land-drops-ready.md (recordsStopped), _lyx/plan/42-orch-role-stencil.md, _lyx/plan/00-overview.md (Shared Decision on the records-stop line)
    summary: Cards 17 and 42 keep a landed line for a landing whose prime records did not pull, but D9 drops the landed notice and acceptance requires that no landed notice be sent, so the line is scope the decision record does not license.
    fix: "Ask the parent, through `lyx loom decision add`, to record an Added-after-Discussion entry deciding whether a records stop on a successful landing still notices the orch. Until the record carries that entry, drop recordsStopped and its test row from card 17, drop the records-stop sentence from card 42 and from card 17's doc lines, and drop the Shared Decision `the records-stop line of a successful landing` from 00-overview.md. If the parent keeps the line, cite the new entry in place of the Shared Decision's rationale."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/23-loomcli-arbiter-delegates.md (**Edit:** arbiterVerb, loomCLI.arbiterDepsFor)
    summary: Card 23 deletes arbiterDeps and arbiterInput, which change the signatures of arbiterVerb and loomCLI.arbiterDepsFor, but lists both without the re-sign arrow.
    fix: "In card 23's **Edit:**, write `internal/loomcli#arbiterVerb` -> `func arbiterVerb(out io.Writer, deps shedarbiter.Deps, in shedarbiter.Input) int` and `internal/loomcli#loomCLI.arbiterDepsFor` -> `func (c *loomCLI) arbiterDepsFor() (shedarbiter.Deps, error)`. Their callers, loomCLI.arbiterCmd and loomCLI.goDriverDepsFor, are already on the card. The plan gate's `--overlay` run accepts both arrows."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/14-innerrun-spawn-hold-yield.md (holdSpawnWhileCovered)
    summary: Card 14 runs the owner re-check only while no pause covers the slug, but D4 says the spawn hold re-checks the owner on each due probe whether or not InnerRun's own owner pause took effect, as today's hold does.
    fix: In card 14's Intent, keep the owner check on a due probe whenever ownedBy stands, covered or not, as today's holdSpawnWhileCovered loop does. Only then yield with the pause reason or the owner reason. Add a TestInnerRun_OwnerCheck row in which an owner pause covers the slug, a probe is due, and an owned-here verdict clears ownedBy while the pass still yields on the pause.
  - id: F9
    severity: LOW
    class: decision
    origin: lens:guard-reach
    location: _lyx/plan/24-arbiterstop-batten-rule.md (Eligibility with Batten), _lyx/plan/18-row-notices-after-chain.md (PauseHonouredPrefix)
    summary: Land's pause-honoured stop is blocked, so under card 24's rule it reaches the tackler and now the arbiter, which can relaunch over the operator's pause request before card 18's silent hand-back.
    fix: Ask the parent through `lyx loom decision add` whether a pause-honoured Land stop is a hand-back under D8. If so, add to card 24 a refusal of a batten stop whose reason starts with battenshed.PauseHonouredPrefix, ending with BattenHandBackWayForward, plus a TestEligibility_BattenRule row reaching it through Check.
  - id: F10
    severity: LOW
    class: decision
    origin: handler
    location: _lyx/plan/34-dobby-owned-elsewhere.md (recheckOwnerPause)
    summary: Card 34 pulls the Bolt level with origin on every probe tick before its owner read, and lifts nothing while origin is unreachable, but the Added-after-Discussion D4 entry reads the effective owner from the local Bolt.
    fix: In card 34's Intent, read the slug's effective owner from the local Bolt as the Added-after-Discussion D4 entry says, and drop the pull seam and the unreachable-origin clause. Alternatively, ask the parent through `lyx loom decision add` to record the pull and its per-tick cost before keeping it.
  - id: F11
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/37-beam-verbs.md (dobbyCmd)
    summary: The new hidden `lyx beam dobby <worktree root>` verb states in its Intent neither what a hand invocation on any path can do nor what bounds it.
    fix: In card 37's Intent, state that dobbyCmd refuses a path DobbyGeometry cannot place as the prime or a beam pair of this hub, naming `lyx beam start`. State that a second invocation exits on the held lock, and that one on a worktree another machine owns exits on card 32's owner read. Add a row for the refused path to TestRegisterDobby or to a dobbyCmd test.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/24-arbiterstop-batten-rule.md (TestEligibility_BattenRule, landing-halt row)
    summary: No batten status can be a landing halt, because Check computes LandingHalt through landingshed.HandsBack, which knows only Merge-In and Finalize, so the batten landing-halt row tests a state a batten run never produces.
    fix: In card 24, drop the landing-halt row from TestEligibility_BattenRule and say that batten has no landing halt. Alternatively, if D8's batten landing halt means a Land stop, name the Land reason that marks it and have Check set LandingHalt for it under Readers.Batten, with the row reaching that reason.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/26-batten-pass-exit-codes.md (TestPassExitCode)
    summary: TestPassExitCode's rows wrap each owned-elsewhere source by the pre-run, but claimBoardEntry's lost claim happens only inside the Worktree-Create row and never at pre-run.
    fix: In card 26's Tests sentence, drop claimBoardEntry from the sources wrapped by the pre-run. Instead, add a row asserting that a row error carrying OwnedElsewhereError leaves the pass's exit at the driver envelope's own code, not 77.
  - id: F14
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/28-batten-first-pass.md (ensureParentDobby)
    summary: A hand pass or a beam land start of a run whose parent another machine owns spawns a parent dobby that exits at once on card 32's owner read, so the run is left undriven with no warning naming `lyx adopt <parent>`.
    fix: In card 28's Intent, have ensureParentDobby read the parent's effective owner, through hubgeom.ParentOf and the board's owners, before it spawns. When another machine owns the parent, return a warning naming `lyx adopt <parent>`, or `lyx adopt --prime` for the prime, and spawn nothing. Add a TestStartFirstPass row for it.
  - id: F15
    severity: LOW
    class: scope
    origin: lens:way-forward
    location: _lyx/plan/07-draft-writers.md
    summary: Refusals whose way forward is a cancel followed by a fresh `lyx batten run`, the left-pair refusal in startGuard.refuse and preDriveCheck's fresh-start text, now lead into the draft refusal, because a cancelled task is left draft.
    fix: "On card 7, add `internal/battencli#startGuard.refuse`, which is already listed, and `internal/battencli#battenCLI.preDriveCheck`, with the release step in their texts: `... then lyx board set-status '{\"slug\":\"<slug>\",\"status\":null}' and lyx batten run <slug>`. Also add every test asserting those texts."
  - id: F16
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/39-adopt-refusals.md (adoptRefusal, named call for a live pair)
    summary: For a reached entry with a live pair, the named call carries the end of the pair's driver, which is no flag or verb, so the named call is refused again at step 1 as typed.
    fix: "In card 39's Intent, for a reached entry whose pair is live, name the step that ends the driver first, `lyx tender pause <slug>`, which makes LivePair read not live, followed by the named call. Add a TestAdopt row asserting that text and that the named call then passes."
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/15-innerrun-respawn-cap.md (conditionText, failed revive)
    summary: The new driver-dead text for a failed revive, `driver strand could not be revived`, names no next step, while the cap text on the same card names `lyx loom start`.
    fix: "In card 15, word the failed-revive text as `driver strand could not be revived; run \"lyx loom resume\" in the task worktree to bring a driver back`. Assert that text in TestNotice_DeadDriverWhileRunning's new row."
  - id: F18
    severity: LOW
    class: scope
    origin: lens:way-forward
    location: _lyx/plan/36-dobby-board-tidy.md (tidyStep timeout)
    summary: The no-report line after board_tidy_timeout_min names no next step, and the card does not say whether the landed slugs since the last tidy carry over to the next tidy.
    fix: In card 36's Intent, state that a timed-out tidy keeps its landed slugs for the next tidy. Word the no-report line to name them and the next tidy, for example `board tidy after <n> landings gave no report; its slugs carry to the next tidy`. Add a TestCountLandingsAndTidy row in which a timeout is followed by the next tidy naming the earlier slugs.
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/28-batten-first-pass.md, _lyx/plan/13-innerrun-short-check.md, _lyx/plan/04-dobby-records.md, _lyx/plan/22-shedarbiter-package.md
    summary: Module docs and PATTERN background files go stale across cards. adoptcli/doc.go and beamcli/doc.go name ResumeInWindow after card 28 deletes it. pattern/PATTERN-shed-driver.md still describes batten's InnerRun wait. pattern/PATTERN-cwd-resolution.md's bound packages list lacks the new told-geometry engines dobbyengine and shedarbiter.
    fix: "Make these edits: card 28 adds `internal/adoptcli/doc.go` and `internal/beamcli/doc.go` under **Edit:** and rewords their ResumeInWindow sentences to the detached first pass. Card 13 adds `pattern/PATTERN-shed-driver.md` and rewords its Outside it paragraph to one check per pass. Cards 4 and 22 each add `pattern/PATTERN-cwd-resolution.md`, appending dobbyengine and shedarbiter to its Bound packages list. The plan gate's `--overlay` run accepts these targets."
  - id: F20
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/36-dobby-board-tidy.md (TidySpec)
    summary: TidySpec's spec names the role, prompt, directives and output file, but no segment or skill list, which PATTERN-parent-directive's spawn-color and role-skill rules put on the launch spec.
    fix: In card 36's Intent, state that TidySpec sets the launch spec's Segment to a segmentcolor segment the beam module names, and its Skills to the skill list the tidy agent needs (`ly:board`, or none, stated). Assert both in a TestTidyPrompt or TidySpec row, as TestArbiterSpec does for the arbiter.
  - id: F21
    severity: LOW
    class: decision
    origin: handler
    location: _lyx/plan/27-dobby-start.md (EnsureLive, prime owner claim)
    summary: Card 27 records this machine as the prime owner when none is recorded, but does not say what EnsureLive does when that owner write fails, as with origin unreachable.
    fix: In card 27's Intent, state that a failed prime claim is reported for the prime with its cause and `lyx beam start` as the re-run, and that EnsureLive starts no prime dobby and goes on with the beams. This follows D2's "as a batten resume does", where reconcileResume warns and goes on. Add a TestEnsureLive row for it.
---

# Round 1 plan review — umbrella-daemon

The plan carries every Decision, Constraint and Added-after-Discussion entry of the decision record to some card; the walk below found no Decision without a card.
Its defects are in how passes start, in what changing InnerRun and the window flags forces outside the cards, and in a handful of bounds and texts.

### [BLOCKING] Detached passes start from the hub root, where batten cannot resolve

**Location:** `_lyx/plan/33-dobby-passes.md` (`SpawnPass`), `_lyx/plan/28-batten-first-pass.md` (`StartFirstPass`)

**Issue:** Card 33's `SpawnPass` starts `<lyx> batten run <slug>` "in its own process group from the hub root".
Card 28's `StartFirstPass` starts the same command "with the hub root as its working directory".
Batten's `resolvePersistentPreRun` resolves its location through `lyxcwd.CwdFrom` and `lyxcwd.Resolve` (`internal/battencli/cli.go:369`, `internal/battencli/arm.go` `arm`).
`Resolve` returns `ErrNotAGitRepo` outside a worktree, and the hub root `/home/knatte/Code/loomyard-LYXHUB` is no repository: `git -C … rev-parse --show-toplevel` answers "not a git repository".
So every pass dobby spawns, and every detached first pass of `lyx adopt` and `lyx beam land`, aborts at arm.
D1's hub-root working directory is the dobby's own.
Today's window run starts batten in the prime's anchor (`engine.OpenWindowIn(name, c.location.AnchorPath(), lyxArgs)`), as orchcli's spawn does (`internal/orchcli/spawn.go:31`).

### [BLOCKING] The forced edits outside the cards are not enumerated

**Location:** cards 13, 28, 4, 35, 36

**Issue:** Several card changes break readers the cards do not list.
- Card 13 makes Run-Shed yield after one check, but battencli's tier1 tests pin the in-call wait.
  `stepStepDrivenRunShed_ReturnsOnTheChildsStateChange` asserts a step that blocks past 1.8 s and ends Stuck (`internal/battencli/lifecycle_integration_test.go:782-856`).
  `stepRunShedPausedChild_WaitsThenTearsDownOnceDone` moves the child through `InnerRun.Sleep`.
  `wireForHub` ends the wait through the pause flag after the first sleep (`:116-128`).
  `stepAwaitingChildDoneTeardown_ArchivesTheRunRecords` asserts `Outcome == Stuck` (`internal/battencli/records_integration_test.go:105`).
- Card 13 writes `innerRunMemory` through `internal/state`, which battenshed's production code does not import today.
  `battenshedAllowedImports` is a closed allowlist, so `TestToldGeometryInvariant_AllowlistOnly` fails (`internal/battenshed/seam_enforcement_test.go:23-31`).
- Card 28 removes `windowFlag` and `openTerminalFlag`, but `battenCLI.registerSeededRun` reads `c.windowFlag` (`internal/battencli/arm.go:296`), which breaks the build.
  Three tests set the fields: `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`, `TestArmSeed_RunAndStepSeedBeforeWire` and `stepCommandRows_FollowTheTasksLocalSwitch`.
- Card 4 pins dobbyengine's production imports with `scankit.AssertImportAllowlistNoStale`, which fails on a stale entry as well as on a missing one (`internal/testkit/scankit/scankit.go:346-369`).
  So the allowlist can hold only card 4's imports.
  Card 35 then imports boardengine for `IsEmptyStatus`, and card 36 imports shuttleengine and stencilstore.
  Neither card edits the allowlist.

The combined `lyx loom validate-plan --overlay` run with these targets added reports no blocking finding beyond the baseline.

### [MEDIUM] A state change between passes is not seen before the new state's arm runs

**Location:** `_lyx/plan/13-innerrun-short-check.md`

**Issue:** `Call` runs `p.observeState(status.State, baseline)` before it builds `childWait` (`internal/battenshed/innerrun.go`, `Call`).
`observeState` replaces the episode when the state differs (`internal/battenshed/notice.go:239-246`).
So a `begun` taken from "the episode's persisted state" after that call already equals the new state.
`checkChild` also returns nil when the file's modification time equals the baseline and no probe is due.
The arm is then picked from the new state, so a child that went from running to done between passes can return Done with no `child running → done` history entry.

### [MEDIUM] Batten's arbiter goto is unbounded in its card

**Location:** `_lyx/plan/25-batten-arbiter-wiring.md`

**Issue:** Card 25 wires "batten's producers and no goto guard" and says the arbiter may make "a goto to one of them".
`shedengine.CheckGotoTarget` admits only rows at or before the current one (`internal/shedengine/goto.go:149-165`), and the card does not state that bound.
Within it, a goto back from Land or Run-Shed re-enters Seed-Child, which rewrites the task worktree's local config layer and seed on every Call (`internal/battenshed/seamchild.go:52`, `:139-143`).
It can also re-enter Worktree-Create over a live pair.
Batten's tackler is bounded to the halted row (`GotoRoutedFrom: false`, `internal/battencli/drive.go` `tacklerDepsFor`).
The attack-surface rule wants the card's Intent to state what the new routing edge lets through and what bounds it.

### [MEDIUM] An undecodable dobby state record has no way forward

**Location:** `_lyx/plan/04-dobby-records.md`, `_lyx/plan/31-batten-pass-wiring.md`, `_lyx/plan/32-dobby-loop.md`

**Issue:** `UpdateState` reads before it writes, and `internal/state` returns `ErrDecode` on a bad record.
So every tick of card 32 fails, logged once per episode, and the dobby drives nothing.
Card 31's `dobbyReading` returns "its read error" into the pass wiring, so hand passes of that parent's runs can fail as well.
Card 13 reads its own undecodable memory record as empty with one Warn, and hand-editing a state file is no way forward under `refusal-spec.md`.

### [MEDIUM] The records-stop line of a successful landing is not licensed by D9

**Location:** `_lyx/plan/17-land-drops-ready.md`, `_lyx/plan/42-orch-role-stencil.md`, `_lyx/plan/00-overview.md`

**Issue:** D9 reads "`landed`: dropped; a landing that succeeds needs nothing from the orch", and acceptance says "no `landed` notice is sent on success".
Card 17 keeps a line `<slug>: landed; the prime's records did not pull: <stop>` on a landing that goes on, and card 42 lists it as "the one notice of a successful landing".
The overview grounds it in a Shared Decision, which is the plan writer's own call.
A scope addition after the Discussion is the parent's to record.

### [MEDIUM] Card 23 re-signs two members without the arrow

**Location:** `_lyx/plan/23-loomcli-arbiter-delegates.md`

**Issue:** `arbiterVerb(out io.Writer, deps arbiterDeps, in arbiterInput) int` (`internal/loomcli/arbiter.go:84`) and `func (c *loomCLI) arbiterDepsFor() (arbiterDeps, error)` (`:459`) both name types card 23 deletes.
The card makes them take and return `shedarbiter.Deps` and `shedarbiter.Input`, but lists both glyphs without the re-sign arrow.

### [LOW] The spawn hold drops the owner re-check under a covering pause

**Location:** `_lyx/plan/14-innerrun-spawn-hold-yield.md`

**Issue:** D4 says that for an owner InnerRun finds inside a continue pass, the spawn hold "re-checks the owner on each due probe", whether or not that owner pause took effect.
Today's `holdSpawnWhileCovered` runs `ownerStep` on a due probe while `ownedBy` stands, covered or not (`internal/battenshed/innerrun_pause.go`).
Card 14 runs it only "while an owner stands and no pause covers", so InnerRun's own owner pause stops every re-check.

### [LOW] An operator's pause at Land now reaches the arbiter

**Location:** `_lyx/plan/24-arbiterstop-batten-rule.md`, `_lyx/plan/18-row-notices-after-chain.md`

**Issue:** Land's pause-honoured Stuck persists blocked (`internal/battenshed/land.go:173-177`), and `decideDrive` sends a blocked stop to the tackler (`internal/sheddriver/driverules.go:233`).
Card 24 admits every blocked batten stop but the three hand-backs, so the arbiter can now relaunch over the operator's pause request.
D8 lists the hand-backs without deciding this case.

### [LOW] Card 34 adds a Bolt pull the Added-after-Discussion entry does not name

**Location:** `_lyx/plan/34-dobby-owned-elsewhere.md`

**Issue:** The D4 entry added after Discussion reads "dobby reads the effective owner from the local Bolt, and when it no longer reads another machine it lifts its own owner pause entry".
Card 34 pulls the Bolt first on every probe tick and lifts nothing while origin is unreachable.
That is a network operation per marked run per tick, and a lift the entry would make offline is withheld.

### [LOW] The hidden dobby verb states no bound

**Location:** `_lyx/plan/37-beam-verbs.md`

**Issue:** `dobbyCmd` takes any worktree-root argument and is "launched only by `SpawnDetached`", but nothing in the card stops a hand invocation.
The card does not say what a hand invocation on any path can do, nor that card 32's lock and owner exits and card 27's geometry refusal bound it.

### [LOW] The batten landing-halt row tests a state batten never produces

**Location:** `_lyx/plan/24-arbiterstop-batten-rule.md`

**Issue:** `Check` sets `LandingHalt` through `landingshed.HandsBack`, which matches only Merge-In and Finalize (`internal/landingshed/handback.go:15-22`).
No batten row has either name, so the row needs a loom producer.

### [LOW] TestPassExitCode models a pre-run claim loss that cannot occur

**Location:** `_lyx/plan/26-batten-pass-exit-codes.md`

**Issue:** `claimBoardEntry` runs only inside the Worktree-Create row (`internal/battencli/owner.go:169-189`), and D4 sends that loss through the shed chain.
A pre-run-wrapped row for it pins no real path.

### [LOW] A parent dobby is spawned where it cannot drive

**Location:** `_lyx/plan/28-batten-first-pass.md`

**Issue:** `ensureParentDobby` spawns the parent's dobby whenever it reads not live, even when another machine owns the parent.
That dobby exits on card 32's owner read, and the hand pass or beam land returns no warning naming the adopt that D2 names for this case.

### [LOW] Cancel-and-restart refusals now meet the draft refusal

**Location:** `_lyx/plan/07-draft-writers.md`

**Issue:** The left-pair refusal says "run `lyx batten cancel <slug> --remove` … then `lyx batten run <slug>`" (`internal/battencli/startguard.go:160`).
`preDriveCheck` says "run `lyx batten run <slug>` for a fresh start" (`internal/battencli/arm.go:711`).
After card 7 the cancelled task is draft, so following either text meets a second refusal first.

### [LOW] The adopt refusal's named call cannot carry a live pair's answer

**Location:** `_lyx/plan/39-adopt-refusals.md`

**Issue:** For a reached entry with a live pair, the card says the named call "also carries … the end of the pair's driver".
`takeOverSlug` stops a live pair under any flag but `--keep-local` (`internal/adoptcli/adopt.go:292-317`), and no flag ends a driver.

### [LOW] The failed-revive notice names no next step

**Location:** `_lyx/plan/15-innerrun-respawn-cap.md`

**Issue:** The cap text on the same card names `lyx loom start`, while the failed-revive text ends at "could not be revived".

### [LOW] A timed-out tidy loses its landings

**Location:** `_lyx/plan/36-dobby-board-tidy.md`

**Issue:** Past `board_tidy_timeout_min` dobby "posts that no report came".
The card does not say whether the landed slugs since the last tidy carry over, so those landings may never be read against the notes.

### [LOW] Docs and PATTERN backgrounds go stale across cards

**Location:** cards 28, 13, 4, 22

**Issue:** `internal/adoptcli/doc.go:45` and `internal/beamcli/doc.go:59` name `battencli.ResumeInWindow` after card 28 deletes it.
They stay stale until cards 39 and 37, against the docs-in-the-same-commit rule.
`pattern/PATTERN-shed-driver.md`'s "Outside it" still says "Batten's InnerRun wait polls a child run" after card 13.
`pattern/PATTERN-cwd-resolution.md`'s "Bound packages" list names the told-geometry engines, but not the two this plan adds.

### [LOW] The tidy agent's launch spec names no segment or skills

**Location:** `_lyx/plan/36-dobby-board-tidy.md`

**Issue:** PATTERN-parent-directive puts a spawned session's color segment and skills on the launch spec.
The arbiter's spec sets both (`internal/loomcli/arbiterprompt.go:101-113`), but card 36's `TidySpec(prompt, reportPath, model, timeout)` names neither.

### [LOW] A failed prime claim in EnsureLive is not decided

**Location:** `_lyx/plan/27-dobby-start.md`

**Issue:** `EnsureLive` claims the prime through `boardengine.SetOwners`, which needs origin.
The card does not say whether a failed claim stops the prime dobby, the beams or the verb.
`lyx orch start` calls this first and must never stop on it.

## Focus verdicts

- Fidelity: D1 to D13, the three Added-after-Discussion entries and every Constraint each have a carrying card, among 1–4, 5–7, 8–16, 17–18, 19–25 and 26–42.
  The only scope without a licence is the records-stop line (F6), plus the pull in card 34 (F10).
- Card order across the dobby chain: no card uses a symbol, exit code or record a later card creates.
  The codes and records are declared in card 4, the start helper in card 27 and the verbs in card 37.
  Cards 27 and 28 reach the `lyx beam dobby` verb only at run time, and their tests fake the spawn.
  The package edges agree with `go list`: battencli imports orchcli, and beamcli and adoptcli import battencli.
- Attack surface: `--continue-pass` and `--no-reopen` (card 30) and the adopt refusals (card 39) state their bounds.
  The arbiter's goto (F4) and the hidden dobby verb (F11) do not.
- Reachability: the refusals of cards 7, 26, 35 and 39 each have a named input and test that reach them, bar the two rows in F12 and F13.
  Card 34 adds handling, not a refusal.
- Shared files with `discussion-scope-in-lenses`: cards 17, 18, 24, 25 and 42 change no Merge-In wording, and no card edits this task's own board entry.
  Card 28 removes another note, `batten-window-default`, which D4 orders.
- PATTERN fit: `shedarbiter` imports `loomengine`, which links only `quarry/glyph` and no tree-sitter (`go list -deps`), so D8's tree-sitter bound holds.
  `sheddriver` gains no `loomengine` import.
  dobbyengine's breaks are in F2's allowlist, F19 and F20.
- Member glyphs and re-sign arrows: cards 13, 16, 18, 25 and 31 re-sign nothing that lacks an arrow; only card 23 does (F7).
- Research steps and verify lines: no card leaves a fact to be found while it runs, and no card carries a `**Verify:**` line.

## Rejected

- guard-reach 7 (tender's `orphanOwner` not widened): between passes a driven run is at Run-Shed or later with its task worktree present, so `orphanOwner`'s worktree-name check keeps it from reading as an orphan.
- guard-reach 9 (a first pass under a pause leaves a seed and a prime claim): that ordering of `armAt` before `battenPreRun` is today's for every hand run, and D4 and D5 name only notices, markers and backoff for a covered first pass.
- guard-reach 5 (the lift trigger waits one probe on the spawn-hold path): D4's bound already accepts each wait ending up to one `notice_probe_s` later.
- way-forward F2 (a beam owned elsewhere is skipped silently in `EnsureLive`): D2 asks for the report of each beam this machine owns and the prime's owner, so a skipped foreign beam is no defect.
