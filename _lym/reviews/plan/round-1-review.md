---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/05-pairteardown-read-run.md:27-36
    summary: ReadRun in pairteardown uses battenrecipe.RowsBeforeRunShed, NameLand and NameWorktreeTeardown, but importing battenrecipe from pairteardown closes a test import cycle (battenrecipe -> preflightshed, whose in-package tests import hubforge, which reaches pairteardown), so the card cannot build its own gate
    fix: "In card 5, add under Create `plan:internal/pairteardown#runShedRow` -> `const runShedRow = \"Run-Shed\"`, declared in followup.go's const block beside landRow and worktreeTeardownRow. Rewrite the Intent so ReadRun names no battenrecipe symbol: BattenStarted is true when batten's row is runShedRow, landRow or worktreeTeardownRow (the rows from Run-Shed on), and ChildLanding and PastParentMerge read landRow and worktreeTeardownRow in place of battenrecipe.NameLand and battenrecipe.NameWorktreeTeardown. Add a TestReadRun row that pins runShedRow to battenrecipe.NameRunShed and the started set to the recipe rows after battenrecipe.RowsBeforeRunShed(), as TestBattenFollowUp pins the other two. State in the Intent that pairteardown cannot import battenrecipe, as followup.go:18-21 records."
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/11-dobby-pass-act.md:3-20
    summary: Card 11 makes dobbyengine import internal/runfsm through RunView's new Run and Verdict fields, but TestToldGeometryInvariant_AllowlistOnly fails any import missing from dobbyengineAllowedImports, and card 11 does not list that allowlist
    fix: "In card 11, add `internal/dobbyengine#dobbyengineAllowedImports` under Edit, and say in the Intent that the allowlist gains `github.com/Knatte18/loomyard/internal/runfsm`."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/06-board-display-from-fsm.md:3-25
    summary: Card 6 changes the board status words but misses part of what asserts or states them; TestBattenIntegration_Rows (tier1) asserts running · Worktree-Create at lifecycle_integration_test.go:1067, and the battenCLI doc (cli.go:119), RunStatus and runStatusSeparator's docs (runlock.go:13,16) and contracts/orchrole/orch-role.md:312 still state the old form
    fix: "In card 6, add under Edit `internal/battencli#TestBattenIntegration_Rows`, `internal/battencli#battenCLI`, `internal/boardengine#RunStatus`, `internal/boardengine#runStatusSeparator` and `contracts/orchrole/orch-role.md`, and add `internal/boardengine/doc.go` under Uses. Extend the Intent: the integration test asserts `starting · Worktree-Create` after the refused row; the battenCLI doc names the claim `starting · Worktree-Create`; RunStatus's and runStatusSeparator's docs call the first half the FSM state; orch-role.md's pre-loom example reads `starting · Worktree-Create`."
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/05-pairteardown-read-run.md:28
    summary: ReadRun reads TaskBranchOnOrigin for every cancelled batten record, but that read is a git ls-remote (fabricengine/remoteleftover.go:236), and card 11 runs ReadRun for every listed run on each dobby tick, so a kept cancel puts a network call on every tick and drops out of the dobby's view offline; card 4 says the FSM's origin reads make no network call
    fix: "In card 4, add under Create `plan:internal/fabricengine#FetchedTaskBranchOnOrigin` -> `func FetchedTaskBranchOnOrigin(prime *lymcwd.Location, slug string) (bool, error)`, which reads slug's task branch from the origin remote-tracking refs the last fetch left, with no network call, and add a TestOriginRunReads row for it. In card 5, add `plan:internal/fabricengine#FetchedTaskBranchOnOrigin` under Uses and fill TaskBranchOnOrigin through it in place of the fabric topology's TaskBranchOnOrigin; add a TestReadRun row showing the read makes no network call."
  - id: F5
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md:46-53
    summary: The shed-pause aside lets lym batten run, step, loom start, loom resume, batten cancel and tender resume act on a run whose shed state is paused, but D1 names only the pause intent, the cancel intent and the hold as asides and has the act rule wait on a shed state paused, so the aside widens D1 with no Added after Discussion entry
    fix: "Ask the parent with `lym creel post --parent --expect-reply` to record the shed-pause aside as an Added after Discussion entry with `lym loom decision add`. The entry should name the verbs that set it aside and the reason the overview's shed-pause-aside decision gives: a shed pause's own resume verb would otherwise refuse itself. Then cite that entry from the shed-pause-aside Shared Decision. If the parent declines, drop ShedPause from runfsm.Aside in card 3, and in cards 7, 10, 12 and 14 word the refusal the record then requires."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-cancel-readers-intent.md:56-61
    summary: Card 8 makes every reader take a cancel as present only when an unfinished intent is in the record, and card 7 writes that intent only after the cancel lock is taken and the Windows check passes, so between the two a start guard, pre-drive check or DobbyRuns reads no cancel while one is under way; today's marker is written inside the lock take (cancelmarker.go:137-159)
    fix: "In card 8, change readCancelMarker so a cancel lock (`shedrun.CancelMarkerLock`) held by another process reads present and writer-alive whether or not the record holds an intent yet, and judgeCancelMarker takes that reading. Add a TestCancelMarker row: a held lock with no intent in the record reads present with a live writer."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/03-runfsm.md:95
    summary: The paused refusal names lym batten run, which sets only ShedPause aside, so a run whose Paused comes from the record's pause intent with no local pause entry (after .lym is removed, or on a machine that took the run over) is refused again by the verb its own way forward names
    fix: "In card 3, have Verdict carry whether Paused came from the pause intent or from a shed state, and word the paused refusal by that: an intent names `lym tender resume <slug>` then the re-run, and a shed pause keeps `lym batten run <slug>` or `lym loom resume`. Add a TestAct row for each wording."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-dobby-pass-act.md:31
    summary: Card 11 keeps drivable as the first gate, but the act rule now waits every batten-paused run (paused-before-landing), so drivable's paused-and-ReachedParent clause, RunView.ReachedParent and DobbyRuns' per-tick reached-parent read can no longer give a pass and are dead
    fix: "In card 11, add `internal/dobbyengine#drivable` under Edit and drop its paused-and-ReachedParent clause, so drivable reads only running. Drop the RunView.ReachedParent field, and drop DobbyRuns' reached-parent read of a paused run with the unreadableReachedParent episode set. Rewrite TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent to assert the act-rule verdict of a paused run whose work reached its parent, with no pass."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-runfsm.md:16,57
    summary: runfsm.Order has no production caller on any card, only TestDerive, so the test-only-code guard (PATTERN-test-only-code) fails it
    fix: "In card 3, state that Derive walks Order() and takes the first state whose condition holds, so Order is reached from production through Derive; otherwise drop the Order target and its use in TestDerive."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/11-dobby-pass-act.md:37-39
    summary: Once card 8 retires the cancel marker file, dobbyengine/doc.go (lines 154, 286, 338, 385) and the lym batten cancel Long text (cancelcmd.go:67, leaves its marker) still describe the marker, and neither card 7, 8 nor 11 names those texts
    fix: "In card 11, extend the doc.go bullet so every paragraph that names the cancel marker names the unfinished cancel intent in the batten record. In card 7, extend the cancelCmd Long-text change so the end-only sentence says it leaves the cancel intent committed locally, not its marker."
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/14-tender-pause-intent.md:37-38
    summary: Resume refuses on the Landed terminal wait, so a slug pause entry on a landed slug whose pair is still here (board done, no batten record) can be lifted only by a hub-wide resume, while the refusal's way forward, lym board list, lifts nothing
    fix: "In card 14, have Resume's Verdict seam treat a terminal wait as act, since a resume lifts an entry and drives nothing, and add a TestResume row: a landed verdict lifts the slug's entry. Otherwise word the terminal refusal there with the hub-wide `lym tender resume`."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-cancel-intent.md:52-54
    summary: The act-rule refusal's named test row feeds a hand-over, an input judgeOrigin already refuses upstream (cancel.go:553-560 and 571-572); the input that reaches the branch is a slug whose board status is done with no batten record, and no row names it
    fix: "In card 7, name the TestCancelRun row for the act-rule refusal as a slug with board status done and no batten record, read as Landed, refusing with the terminal wording; keep the hand-over row only as an extra case."
  - id: F13
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-cancelled-record-cleanup.md:30
    summary: Card 9 gives a nil PairStands a meaning (a pair standing here), a state only a hand-built test cleanupDeps has, since fabricCleanupDeps always fills it
    fix: "In card 9, drop the nil-PairStands branch, and have every test-built cleanupDeps that reaches a cancelled record set PairStands."
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/09-cancelled-record-cleanup.md:52
    summary: The ImpactSummary says only lym fabric remove reaches the post-removal deletion, but Teardown.Run is also called by the origin-reconcile teardown (pairteardown/originseams.go:179), so that path also deletes a cancelled run's record
    fix: "In card 9, rewrite the ImpactSummary to say the step is reached by every Teardown.Run caller, lym fabric remove and the origin-reconcile teardown, and add a TestTeardown_CallSequence assertion or Intent line saying the reconcile's removal of a cancelled pair deletes its record too, as D3 asks of every pair removal."
  - id: F15
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/12-loom-resume-act.md:18-31
    summary: D1 has every verb that acts on a run ask the act rule, but card 12 guards only lym loom start and resume and leaves the driving verbs lym loom run and lym loom step untouched with no stated reason, unlike card 10, which gives one for lym batten goto
    fix: "In card 12, either have the run and step verbs' pre-run call refuseByAct too, with a TestResumeVerb-style row each, or state in the Intent why they are exempt (role-audience primitives an external supervisor drives after start or resume asked) and what bounds that."
---

The plan carries D1 and D3 faithfully overall, and it defers to the later parts the work the decision record's Delivers line leaves to them.
Two cards cannot pass their own gates as written: card 5's import of the batten recipe package, and card 11's import of runfsm into dobbyengine.

### [BLOCKING] Card 5 imports battenrecipe into pairteardown, which closes a test import cycle

**Location:** `_lym/plan/05-pairteardown-read-run.md:27-36`
**Issue:** `go list` shows the chain: `battenrecipe` depends on `preflightshed`, and `preflightshed`'s in-package tests import `hubforge`, which reaches `internal/pairteardown`.
`internal/pairteardown/followup.go:18-21` already records this cycle and keeps `landRow` and `worktreeTeardownRow` as local constants for that reason.
The overview's runfsm-imports decision says callers fill the row facts, but card 5 places that filling inside pairteardown itself.

### [BLOCKING] Card 11 misses dobbyengine's import allowlist

**Location:** `_lym/plan/11-dobby-pass-act.md:3-20`
**Issue:** `internal/dobbyengine/seam_enforcement_test.go:20-36` holds the closed list of imports dobbyengine may use, and `TestToldGeometryInvariant_AllowlistOnly` fails on any other.
`RunView` gains runfsm-typed fields, so dobbyengine imports `internal/runfsm`, which the list lacks.

### [MEDIUM] Card 6 misses part of the board-status affected set

**Location:** `_lym/plan/06-board-display-from-fsm.md:3-25`
**Issue:** `internal/battencli/lifecycle_integration_test.go:1067` asserts `running · Worktree-Create` after a refused row, so the claim change fails this tier1 test.
The `battenCLI` doc at `internal/battencli/cli.go:119`, the docs of `RunStatus` and `runStatusSeparator` at `internal/boardengine/runlock.go:13,16`, and `contracts/orchrole/orch-role.md:312` still state the old words.
The proposed targets pass `lym loom validate-plan --overlay`.

### [MEDIUM] The cancelled-record branch read makes a network call on every dobby tick

**Location:** `_lym/plan/05-pairteardown-read-run.md:28`
**Issue:** `Topology.TaskBranchOnOrigin` goes through `firstOnOrigin` to `remoteHeadTip`, which runs `git ls-remote` (`internal/fabricengine/remoteleftover.go:236`).
A kept cancel keeps its batten record until `--remove`, so the dobby lists it on every tick, and card 11 runs `ReadRun` per listed run.
Card 4 says the FSM's origin reads use only the remote-tracking refs the last fetch left; this read breaks that, and offline its error leaves the run out of the dobby's view.

### [MEDIUM] The shed-pause aside is not licensed by the decision record

**Location:** `_lym/plan/00-overview.md:46-53`
**Issue:** D1 lists `paused` as a pause intent or a shed state `paused`, has the act rule wait on it, and names only the pause intent, the cancel intent and the hold as asides.
The plan's reasoning is sound: without the aside, `lym batten run <slug>`, the verb that resumes a shed pause, would refuse itself.
But it is a widening of D1 that the record does not carry, and the rubric admits no card scope beyond the answer key.

### [MEDIUM] A cancel under way is invisible until its intent is committed

**Location:** `_lym/plan/08-cancel-readers-intent.md:56-61`
**Issue:** Today `takeCancelMarker` writes the marker within the lock take (`internal/battencli/cancelmarker.go:137-159`).
After cards 7 and 8, the lock is taken, the Windows live check runs, and only then is the intent written and committed.
Every reader keys presence on the intent, so in that window a start or a seed passes the cancel guard.

### [MEDIUM] The paused refusal names a verb that cannot lift an intent-derived pause

**Location:** `_lym/plan/03-runfsm.md:95`
**Issue:** `lym batten run` sets only `ShedPause` aside, so a `Paused` read from the record's `pause_intent` stays `Paused` under it.
D3 makes the pause record a cache, so the intent without a local entry is a real state: a removed `.lym`, or a machine that took the run over before D6's catch-up.

### [MEDIUM] drivable's paused-with-work-on-parent clause is dead under the act rule

**Location:** `_lym/plan/11-dobby-pass-act.md:31`
**Issue:** `internal/dobbyengine/passes.go:71-73` admits a paused run whose work reached its parent, and `internal/battencli/dobbyreads.go:203-216` reads that for each paused run on every tick.
Card 11 gates `planPass` on an act verdict with nothing set aside, which waits every batten-paused run, as the paused-before-landing Shared Decision intends.
The clause and its read can no longer give a pass.

### [MEDIUM] runfsm.Order has no production caller

**Location:** `_lym/plan/03-runfsm.md:16,57`
**Issue:** No card's Uses or Intent calls `Order` from production code; only `TestDerive` reads it.
PATTERN-test-only-code fails a function that no `main` package reaches.

### [LOW] Cancel-marker prose survives the marker's retirement

**Location:** `_lym/plan/11-dobby-pass-act.md:37-39`
**Issue:** `internal/dobbyengine/doc.go` names the cancel marker at lines 154, 286, 338 and 385, and the cancel verb's Long text says it leaves its marker (`internal/battencli/cancelcmd.go:67`).
Card 11 edits that doc.go only for its pass-planning paragraph, and card 7 changes that Long text only for the records removal.

### [LOW] A slug pause entry on a landed slug cannot be lifted by its own resume

**Location:** `_lym/plan/14-tender-pause-intent.md:37-38`
**Issue:** With `Pause`, `Cancel`, `ShedPause` and `PushOwed` set aside, a `Landed` reading still waits as terminal, so `lym tender resume <slug>` refuses and lifts nothing.
`lym tender pause <slug>` asks no verdict, so such an entry can be written while the pair still stands.

### [LOW] The named row for the cancel's act-rule refusal is rejected upstream

**Location:** `_lym/plan/07-cancel-intent.md:52-54`
**Issue:** `judgeOrigin` refuses an owned-elsewhere slug before the act rule is asked (`internal/battencli/cancel.go:553-560` and `571-572`), and the reached-parent check refuses a batten `done` record first.
The input that reaches the new refusal in production is a board `done` slug with no batten record.

### [LOW] Card 9 gives a nil PairStands a meaning

**Location:** `_lym/plan/09-cancelled-record-cleanup.md:30`
**Issue:** `fabricCleanupDeps` always fills `PairStands`; the nil default serves only hand-built test deps.
The code-quality rule says not to handle a state the code's own guarantees rule out.

### [LOW] Card 9's ImpactSummary undercounts the reach of the deletion

**Location:** `_lym/plan/09-cancelled-record-cleanup.md:52`
**Issue:** `internal/pairteardown/originseams.go:179` calls `t.Run`, so the origin-reconcile teardown also passes through the new post-removal step.
The behaviour is what D3 asks, but the summary says `lym fabric remove` alone reaches it.

### [LOW] Card 12 leaves lym loom run and step unasked without a stated reason

**Location:** `_lym/plan/12-loom-resume-act.md:18-31`
**Issue:** `internal/loomcli/cli.go:314,342` define the run and step verbs, which drive the child run.
D1 has every verb that acts on a run ask the act rule; card 12 says the other loom verbs are untouched but gives no reason, where card 10 does for `lym batten goto`.

## Focus items settled

- Card order: no card uses a symbol a later card creates. Card 6 uses only card 3's `TransitionState`, not card 5's `ReadRun`, and card 13's `PullBolt` precedes card 14.
- Fidelity: every D1 and D3 decision this part's Delivers line names is carried by some card. D11's dobby teardown, D6's catch-up, D2's owed push, D4's code tip and D7's holds are the later parts' by the Delivers line. Card 9 carries the cancelled-record deletion for `lym fabric remove` and cleanup. Card 11's paused-before-landing follows D1's order and is no defect. Card 16 puts the attempt counts under `loomengine.LoomDurableDir`, which `statusCommitPathspec` stages on every transition. The one unlicensed widening is F5.
- Attack surface: cards 3, 7, 10, 12 and 14 each state what their aside lets through. The shed-pause aside lacks a licence (F5), not a bound.
- Reachability: as F6, F7, F11 and F12 report. `FinishCancel` with no intent is a fail-loud wiring guard and takes no reaching test.
- Member glyphs and re-sign arrows: the Edit cards over Go files list members, and the changed signatures (`judgeStatus`, `planPass`, `heldCancelMarker.release`, `cancelHold.finishMarker`) carry the arrow. No finding.
- Test economy: card 3's tests are a new package's own tests. Card 5's `TestReadRun` drives a new reader, and its cache-removal test is D3's acceptance criterion, which no existing test covers. Card 8 rewrites existing tests and adds none. No duplicate.
- Board entry: no card writes this task's board entry. Card 6 changes only the run-status words the batten and loom writers compose.
- PATTERN fit: card 17's text matches the declared `PATTERN-lym-files` change, and card 3 records runfsm's import set in `PATTERN-leaf-packages`. Card 4 keeps both origin reads in fabricengine. The adopt pause is still lifted by `lym tender resume <slug>`, as `PATTERN-dobby-runs` says.
- Research steps: no card leaves a design-bearing fact to be found while it runs.

## Rejected

- guard-reach F4, the part about a kept `Cancelled` record: `lym tender resume <slug>` sets `Cancel` aside, so the record no longer reads `Cancelled` and the verdict is act. Only the `Landed` half is kept, as F11.
- affected-set's note on `battencli/drive.go:164-169`: the fork itself showed that state is unreachable, and it raised no finding.
- repo-invariants finding 3 and affected-set findings 1 and 2 describe the same card 6 gap. They are kept once, as F3.
