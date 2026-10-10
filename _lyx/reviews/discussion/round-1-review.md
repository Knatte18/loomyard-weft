---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D3's five-rejection bound on the PR-Gate to Darn edge is false, since every Darn Done ends PR-Gate's bounce episode and max_bounces 5 never trips
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:154
    summary: D10 clears the claim on board state alone, so a landed run whose done mark failed reads as unstarted and a live batten run loses its lock, with no stated bound
  - id: F3
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md:145
    summary: D9's Stuck text in landingshed names the parent weft and D11's loomcli halt note lists paths per side, against PATTERN-fabric-vocabulary and the record's own vocabulary constraint
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:126
    summary: D7 refuses an empty verify only at lyx loom start, while the told reader runs at every call and an empty command skips the darn gate, Publish and Finalize verify with a warning
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:81
    summary: D4 renders the rejection record into the Darn spawn but never says who clears it, a step loom's PR-Rework performs
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D4 does not state that the verify budget resets on each Darn spawn, and D8's claim that twenty logs cover one run rests on a per-run total the record never bounds
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:218
    summary: D12 and D6's tmux-tier confirmation, the subject of Auto-mode assumption 8, trace to no acceptance criterion
---

### [BLOCKING] D3's rejection bound does not hold

**Location:** `_lyx/discussion/decision-record.md:73` (D3, "Routing edge and its bound").

**Issue:** D3 states that the `PR-Gate` to `Darn` edge is "bounded by the five-rejection budget, after which `PR-Gate` halts for a human exactly as in loom".
The engine does not count it that way.
`episodeStuckCount` in `internal/shedengine/run.go` ends a producer's episode at a Done by any producer sharing its non-empty segment (`segmentEnders`), and D3 puts `Darn` in `PR-Review` so the edge validates.
Every rejection therefore runs `PR-Gate` Stuck, `Darn` Done, Publish, `PR-Gate`, and the `Darn` Done resets `PR-Gate`'s count to zero, so the count never passes one and `max_bounces: 5` never trips.
Loom has the same property through `PR-Rework`, which also sits in `PR-Review` and returns Done, so "exactly as in loom" is literally true but describes no bound.
The actual bound is that each re-spawn needs one operator `lyx loom reject`, and the record never states it.
That leaves the attack-surface question unanswered for this edge: a plan writer pinning "the sixth rejection halts" writes a test that cannot pass.

**Fix:** Restate D3's bound as what the engine enforces: each traversal of the edge is one operator rejection, with no mechanical cap, because a `Darn` Done ends `PR-Gate`'s episode.
Drop the claim that `max_bounces: 5` bounds the path, or else make a design change that keeps the count across `Darn` Dones and state that change as a Decision.

### [BLOCKING] D10's clearing condition lets through a landed run and a live batten run

**Location:** `_lyx/discussion/decision-record.md:154` (D10), with the Open risk at line 214.

**Issue:** D10 clears the status when "the entry is not done and its status has the run form", judging only the board.
Two cases pass that test that D10 does not intend, and the Decision bounds neither.

1. A landed run whose done mark failed.
   `Finalize.markTaskDone` in `internal/landingshed/finalize.go` only logs a failed `MarkTaskDone`, and loom's `boardStatus` writes nothing at `done`, so the entry keeps a run-form status after the landing.
   Batten's `Worktree-Teardown` row runs only after a done child, per `contracts/recipes/batten-recipe.yaml`, so on batten's ordinary route this landed-but-unmarked entry is the main case D10 fires on.
   It clears the entry to empty, the state of a task never started, so the board offers the task for another run.
   That contradicts D10's own title, "an unlanded run's board claim".
2. A run removed by hand while its batten run in the prime is still alive.
   `lyx fabric remove` refuses only a busy driver (`RefuseWhenBusy`) and ends a parked or halted one, per `internal/pairteardown/doc.go`.
   The batten run's status under the prime's `_lyx/shed/<slug>/` stays non-done, but D10 releases the lock anyway.
   `RunLockedError` in `internal/boardengine/runlock.go` says to clear the status only after `lyx batten status` refuses, reports `done`, or reports a halted run the operator has decided to abandon.
   D10 skips that check, and the Open risk at line 214 names only the lost trace, not the released lock on a run that can still resume.

**Fix:** In D10, state what the clearing may let through and bound both cases, for example as follows.
A teardown whose run reached `done` does not clear the entry: it leaves it alone or marks it done.
The claim is cleared only when the batten run for the slug is absent, done, or halted.
Then add the matching acceptance criteria.

### [BLOCKING] D9 and D11 name fabric sides in packages outside the owner set

**Location:** `_lyx/discussion/decision-record.md:145` (D9), `_lyx/discussion/decision-record.md:163` (D11), against the Constraints entry at line 196.

**Issue:** `PATTERN-fabric-vocabulary` admits the bare words weft and warp only in its owner set (`fabricengine`, `fabriccli`, `weftname`, `gitkit`, `hubforge`, `boardengine`, `configsync`), and `TestEnforcement_FabricVocabulary` enforces it.
D9's Stuck is produced by Finalize in `internal/landingshed`, and D9 says it names "the parent weft".
D11's section, written by `internal/loomcli/halt.go`, lists dirty paths "per side", and labelling those sides puts side names in a non-owner package.
The record's own Constraints entry says `landingshed` keeps naming "the parent pair" and "the task worktree, never a side".
D9 and D11 contradict that entry and plan a violation of a listed constraint.

**Fix:** Word D9's Stuck as naming the parent pair, with the way forward `lyx fabric commit` run in the parent pair.
Have D11's section either present the dirty paths without side labels, or get any side-labelled text from an owner-set export that renders it whole, and state which one.

### [MEDIUM] D7's empty-verify refusal guards only the start

**Location:** `_lyx/discussion/decision-record.md:126` (D7), with D6 at line 106.

**Issue:** D6 makes the darn verify command a told reader of `darn.yaml`'s `verify`, read at each call by the darn gate, Publish and Finalize.
D7 refuses an empty `verify` only in `lyx loom start`.
An empty command at a later call is not refused: `landingshed`'s verify gate logs "post-merge verify skipped" and passes (`internal/landingshed/verifygate.go`), and D7's own rationale says the darn gate would pass on a warning.
The file is hub-wide, so a single `lyx config darn --set verify=` turns off every safety net of every darn run in flight, including runs that resume or step through `loomcli.ArmAt` without passing `start`.
D7's rationale calls that gate the recipe's "only safety net", yet the guard covers only one entry point.

**Fix:** In D6 and D7, state that the darn verify reader returns an error, not `""`, for an empty or missing `verify`, so the gate, Publish and Finalize each refuse or stop at call time.
Then add the matching acceptance criterion.

### [MEDIUM] D4 leaves the rejection record's lifecycle open

**Location:** `_lyx/discussion/decision-record.md:81` (D4's rejection-findings token).

**Issue:** D4 renders "the content of the rejection record when one exists for the run" into each `Darn` spawn and tells the agent to treat it as that spawn's work.
In loom, `PR-Rework` consumes the record through `ClearRejection` (`internal/loomcli/wiring.go`).
D4 names no row that clears it in darn.
A record left in place keeps `rejectionPending` true, which changes what `statusCommitPathspec` commits.
It also renders already-fixed findings into any later `Darn` spawn, such as one reached by a goto after a Publish failure, and the stencil treats those findings as the spawn's work.

**Fix:** Decide in D4 when the record is cleared, for example by the `Darn` commit decorator on Done of a spawn that rendered it, and add the matching acceptance criterion.

### [LOW] The verify budget per spawn and the log retention count are not reconciled

**Location:** `_lyx/discussion/decision-record.md:83` (D4), `_lyx/discussion/decision-record.md:140` (D8).

**Issue:** The shuttle gate counts failures per shuttle run (`run.gateFails` in `internal/shuttleengine/wait.go`), so each `Darn` spawn starts with a full `verify_attempts` budget, and a halted `Darn` resumed by the operator does too.
D4 does not say so, which leaves part of the D3/D4 focus question open: the verify runs on the path number up to the initial spawn plus one per rejection, times one more than `verify_attempts`.
D8's rationale, "Twenty covers every verify of one run with room", rests on a per-run total the record never bounds.
With the template default of 3 attempts, a run with a handful of rejections exceeds twenty, plus Publish's and Finalize's verifies.
The evidence that #502 is about stays safe because the failure record pins its log, so this is a premise error, not a loss.

**Fix:** State in D4 that the budget is per spawn.
Reword D8's rationale to justify twenty on the failure record's pin, not on covering a whole run.

### [LOW] Two choices trace to no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:218` (Acceptance criteria).

**Issue:** D12's orch stencil, `ly:board` skill and board `recipe` help changes appear in no criterion, although the "Docs and guards" group covers D12's peer doc changes.
D6's `tmux`-tier confirmation for a `publish_verify` failure, which Auto-mode assumption 8 names, also has no criterion.
The "Verify" group covers only the record's failing tests.

**Fix:** Add one criterion for D12's text, and extend the failure-record criterion to the `tmux`-tier confirmation.

## Focus verdicts not raised as findings

- D9's sweep: `CommitRecordsPaths` takes the weft write lock, and status files are written atomically (`fsx.AtomicWriteBytes`), so the commit sweeps in only whole, durable sibling records.
  Batten's next commit over them is a no-op.
  The guard stays strict elsewhere, and the attribution cost is an Open risk.
  The only defect is the vocabulary in F3.
- D6 and D7 trust class: matching `publish_verify`'s hub-wide trust class is sound.
  D6's behavior while a Publish failure record is present is stated as a Decision, not left as an assumption.
  Assumption 8 points at it correctly.
  The gap is F4.
- D8's pinning exception is bounded.
  `PublishFailure` is a single record per verify directory naming one log, so a stale record pins at most one log beyond the newest twenty.
- Completeness before leanness: no requirement a Decision depends on lives only in `support-log.md`.
  Each Interview answer and ledger row resolves to a Decision or an Open risk that already carries it.
