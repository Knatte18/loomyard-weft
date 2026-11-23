---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/webstercli/reset.go:177
    summary: A reset whose state.json bytes do not change lands no records commit, so no Run-Transition is written and the run's last transition still names the dropped code tip, the defect the change exists to remove; the new test passes only because its fixture leaves state.json uncommitted.
    fix: "Make the reset's transition commit always land. In internal/fabricengine, have CommitWithTrailers force an empty weft commit when trailers are given and the warp is born (set forceEmptyCommit in commitWeftLocked when len(trailers) > 0 && !unborn, routing the no-op paths through an empty commit that carries the message with Warp-SHA and the trailers), and state that in CommitWithTrailers' doc comment. In internal/webstercli/reset_test.go, make TestResetCmd_RecordsTheRunTransitionAtTheNewTip commit the seeded state.json to the records side before the reset (the fixture's saveState leaves it dirty today), with a batch whose Recoveries is 0 and RecoveryStartSHA and PreFixHead are empty, so the test fails on the current code; add a row for --to start as well."
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/sheddriver/drivehandback.go:62
    summary: The new watched run-id line rests on a premise batten's code contradicts, since stopHold ends batten's hold at the driver's park marker, at an arbiter hand-on, and at once when no spawns or attempt remain; every watched judged blocked stop now gets two parent notices, and the f7c7933 Root-Cause trailer names a hold that does not keep the notice back.
    fix: "Correct the driveNotices doc comment in internal/sheddriver/drivehandback.go and the three added lines in internal/sheddriver/doc.go to state what the code does: the driver sends the run-id line at a watched blocked stop an agent judged, and batten still sends its own state-changed notice once the run parks, so the parent hears both. Name in landing/summary.md's Design decisions that both notices go at every such stop, replacing the narrower claim that both go only while batten's hold ends without the arbiter's hand-on. If the duplicate is not wanted, ask the run's parent (lym creel post --parent) whether batten's notice or the driver's line should be the one sent, rather than choosing in code."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/sheddriver/drive.go:250
    summary: judged outlives the stop it describes, because it resets only when attempt is empty and never when the stop in hand changes key, so a later stop no agent judged can send the run-id line; no test kills the mutation survivor at drive.go:250.
    fix: "In goDriver, record the key of the stop the agent judged (for example judgedKey StopKey set beside d.judged = true in settleStop), and in handBack set h.Judged only when stop.Key() equals that key; keep the reset at drive.go:250. Update the judged field's doc comment to match. Add a TestGoDriver_Settle row in internal/sheddriver/drive_test.go: watched, an agent judges stop A and the run resteps, then a blocked stop B at another producer hands back by rule; want wantNotifies 0."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/webstercli/reset.go:177
    summary: "The board entry asks that `webster reset` (and `goto`) commit a transition naming the new tip; the diff changes reset only, and the decision record holds no Added after Discussion entry narrowing that, only the writer's own summary argument."
    fix: "Either make lym loom goto's records commit carry Run-Transition naming the run (internal/loomcli, the goto verb's status commit), or leave goto unchanged and ask the run's parent through lym creel post --parent to record the narrowing with lym loom decision add, quoting summary.md's reason that goto never moves the code tip."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-run-fsm.md:21
    summary: "webster reset is now a writer of Run-Transition, which lastTransitionCommit and ReturnToPreviousTransition read as a run transition, but the commit goes through fabricSync's detached push outside statuscommit; PATTERN-run-fsm's background still says a transition takes statuscommit's admission and pushes records then code before the run acts, and nothing records this writer or its push guarantee."
    fix: "In pattern/PATTERN-run-fsm.md, under 'A transition pushes before the run acts', add that webster reset marks a transition at the code tip it moved to through its records commit, and state how that commit reaches origin before a resume reads it. If the detached push cannot give that guarantee, push the records synchronously in reset.go after fabricSync (through fabric's records push) before reporting ok, and say so in the reset doc lines."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-checks-spec.md:287
    summary: The pinned plan-checks spec lists the done-card-edited context check with what raises it, but the diff adds the sibling blocking check overview-frame-edited with no spec entry.
    fix: "Add an overview-frame-edited entry beside done-card-edited in contracts/specs/loom-plan-checks-spec.md's 'Run state and parts' section: raised when the run record holds a begun batch and a recorded overview frame and 00-overview.md's frame outside its Card Index no longer hashes to it; way forward: restore the overview and carry the change in a follow-up card after the last begun batch, with its Card Index line."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomshed/gates_test.go:733
    summary: No TestNewPlanGate row edits only the Card Index after a begun batch and expects a pass, so a gate hashing the whole overview would refuse what Webster's rebaseline accepts with every row still green.
    fix: "Add a row to the overview-frame loop in TestNewPlanGate that, with a begun batch and a recorded frame, changes only a Card Index line of 00-overview.md (for example a reworded one-line intent of a card) and wants Passed true and no overview-frame-edited finding."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/webstercli/wiring.go:217
    summary: Nothing tests that wireHub fills c.runID; the reset fixture injects runID directly, so dropping the wiring line would lose the trailer in production with every test green.
    fix: "In an existing hub-mode TestWire_* test in internal/webstercli (for example TestWire_ModeHubSelectsHubMode), assert that the wired websterCLI's runID equals the location's WorktreeName."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/gates.go:319
    summary: With no PlanOverviewFrameHash recorded the gate reports nothing, while Webster's rebaseline falls back to the baseline copy's frame or refuses any overview change outright, so a record from before frame hashes still passes the gate and blocks at Webster.
    fix: "In overviewFrameEditedFindings, when st.PlanOverviewFrameHash is empty and a batch has begun, treat a 00-overview.md whose hash differs from st.PlanFileHashes[runplan.PlanOverviewFile] as edited (the rebaseline refuses every overview change then), and add a TestNewPlanGate row for it; or state in the function's doc comment that such records are left to the rebaseline."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/gates_test.go:733
    summary: The documented contract that an unreadable overview is a returned error from ValidatePlan has no test.
    fix: "Add a TestNewPlanGate row with a begun batch and a recorded frame that makes 00-overview.md unreadable through permkit.Chmod and wants the gate to return an error."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/commit_integration_test.go:223
    summary: commitWeftLocked's doc claims the caller's trailers are written on an unborn warp too, and no test covers that.
    fix: "Add a TestCommitWithTrailers subtest on an unborn-warp fixture asserting the weft message ends with the caller's trailer and holds no Warp-SHA line."
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/sheddriver/drivehandback_test.go#TestDriveNotices
    summary: The redundancy report lists TestDriveNotices as contained by TestGoDriver_Settle, but it pins the exact notice text and the judged-by-state matrix, which TestGoDriver_Settle checks only as notice counts.
    fix: "Add the line //testtiming:keep exact hand-back notice text and the judged-by-state matrix directly above func TestDriveNotices."
---

### [BLOCKING] A reset with unchanged state.json writes no transition

**Location:** `internal/webstercli/reset.go:177`, with `internal/fabricengine/recordsgit.go` `commitWeftLocked`.

**Issue:** `fabricSync` now calls `CommitWithTrailers`, which inherits `commitWeftLocked`'s no-op rule.
Only snapshot tags set `forceEmptyCommit`, so when the staged pathspec is unchanged, the function returns `("", false, nil)` and the trailers are dropped.
The reset's only change to the records is `PreFixHead = ""`; for `batch-start` it also zeroes `Recoveries` and clears `RecoveryStartSHA`.
`SaveState` is a plain `state.WriteJSON` with no timestamp.
So a reset whose record already holds those values commits nothing, after `ResetPairCode` has already moved the code branch and origin.
Two examples: `--to batch-start` on a batch that died without a recovery, and `--to last-batch-head` with no pre-fix head.
That is the incident's own shape.
The run's last transition then still names the dropped tip, and the resume refusal and the `CodeTipGrace` return come back.
`CommitEmptyWithTrailers` exists for exactly this reason ("so the trailers have a commit to sit on").
The orch's manual fix in the issue was an empty records commit too.
`TestResetCmd_RecordsTheRunTransitionAtTheNewTip` does not catch this: `resetFixture.saveState` writes `state.json` without committing it, so the reset's commit lands on the fixture's own change.

### [MEDIUM] The watched notice rests on a hold batten does not keep

**Location:** `internal/sheddriver/drivehandback.go:62`; also `internal/sheddriver/doc.go` and commit `f7c7933`'s `Root-Cause:`.

**Issue:** The comment says batten holds its own notice of a judged stop, "so the parent would otherwise hear nothing".
The trailer says the same thing.
`battenshed/stophold.go` `stopHold` contradicts both: it ends the hold at a park marker at or after the episode start, at an attempt's hand-on, and at once when no spawns are left and no attempt exists.
`notice.go` `arbiterHolds` holds only while an arbiter or tackler is alive in flight.
`handBackRun` always parks a blocked stop, so batten's state-changed notice still goes out after the driver's hand-back.
With this diff, every watched judged blocked stop sends the parent two notices, including the two `TestGoDriver_Settle` rows the diff adds.
The summary states a narrower overlap than that.
The record says the incident's own records were not available, so the path that actually stayed silent is still unknown.
The run's parent confirmed the route, so this finding asks for the text to match the code and for the duplicate to be decided, not for a revert.

### [MEDIUM] judged outlives the stop it describes

**Location:** `internal/sheddriver/drive.go:250`

**Issue:** `judged` resets only when `attempt == ""`.
After an arbiter resume, the next step runs with a non-empty attempt, so a different blocked stop that hands back by rule inherits `judged` true.
`settleStop`'s `stop = moved; continue` keeps it for a moved stop the same way.
The field's doc comment says "the stop in hand", and the code does not hold to that.
No test kills the mutation survivor at `drive.go:250`.

### [MEDIUM] goto's transition left out without a recorded narrowing

**Location:** `internal/webstercli/reset.go:177`

**Issue:** Defect 3 of the board entry says "`webster reset` (and `goto`) should commit a transition naming the new tip".
The diff changes reset only.
`summary.md` argues that goto never moves the code tip, but that is the writer's claim, not an `Added after Discussion` entry or a correction from the parent.
The argument is plausible, so recording it is enough.

### [MEDIUM] PATTERN-run-fsm does not know reset's transition

**Location:** `pattern/PATTERN-run-fsm.md:21`

**Issue:** The reset's records commit now marks a transition for every `Run-Transition` reader.
It is pushed by fabric's detached push, not through `statuscommit`.
A failed detached push leaves origin's records at the old transition while origin's code branch has moved, so the resume refusal comes back.
The pattern background describes transitions as statuscommit-admitted and pushed before the run acts, and names no other writer.
I judged this a stale background with an unstated push guarantee, not a clear breach of the constraint's declared text: the reset does commit and starts a push before any resume.

### [MEDIUM] overview-frame-edited missing from the plan-checks spec

**Location:** `contracts/specs/loom-plan-checks-spec.md:287`

**Issue:** The spec is pinned and lists each check with what raises it.
Its sibling `done-card-edited` is listed, and the new check is not.

### [MEDIUM] No row shows a Card-Index-only edit passing

**Location:** `internal/loomshed/gates_test.go:733`

**Issue:** The rows cover a framing edit, no edit, no batch and no frame.
None shows that the gate compares the frame rather than the whole overview, which is the half of "agree with the rebaseline" that keeps a legitimate follow-up card passing.

### [MEDIUM] wireHub's runID is untested

**Location:** `internal/webstercli/wiring.go:217`

**Issue:** The reset test injects `runID: loc.WorktreeName`, so the production wiring that the whole reset fix depends on has no test.

### [LOW] Gate stays silent on records with no frame hash

**Location:** `internal/loomshed/gates.go:319`

**Issue:** `recordedOverviewFrame` in `websterengine/rebaseline.go` falls back to the baseline copy's frame.
With no frame at all, `overviewRefusal` refuses any overview change.
The gate returns nothing in both cases, so the two still disagree for such records.

### [LOW] Unreadable overview path untested

**Location:** `internal/loomshed/gates_test.go:733`

**Issue:** The doc comment of `overviewFrameEditedFindings` promises a returned error, and no row reaches that path.

### [LOW] Trailers on an unborn warp untested

**Location:** `internal/fabricengine/commit_integration_test.go:223`

**Issue:** The `commitWeftLocked` doc now claims trailers are written "on a born and an unborn warp alike", and only the born case is tested.

### [NIT] TestDriveNotices needs its keep mark

**Location:** `internal/sheddriver/drivehandback_test.go#TestDriveNotices`

**Issue:** The redundancy report lists it as contained, but its assertions pin the exact line text and the state-by-judged matrix that the covering test does not check.

## Rejected

- Correctness lens F2, the half saying the gate is silent before any batch began: the board entry scopes defect 2 to "when batches have already begun", so not refusing before a begun batch follows the answer key. The no-frame half is kept as F9.
- Tests lens, reset's start and archive-only paths untested: folded into F1's fix, which adds a `--to start` row.
- Correctness lens F4: merged into F2, which keeps its evidence and raises it to MEDIUM because the same false premise is in the commit's `Root-Cause:` trailer and in the summary.
- `Root-Cause:` trailers of `87c375c` and `2154de5`: each names the defect its diff removes (the reset's records commit had only a `Warp-SHA` trailer, and `ValidatePlan` never compared the overview frame). No finding.
