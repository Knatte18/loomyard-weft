---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D5, D8
    summary: The run-exit whole-session audit cross-check is not addressed, so every finding D5 downgrades or D6 dispositions re-fails the run at exit; D5 and D8 also name recover-batch as an audit site, but it runs no audit
  - id: F2
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D4
    summary: The parent-write path split overlaps and leaves lyx run state as policy class, because the reports dir and state.json sit under the git-excluded _lyx link
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D11
    summary: goto's segment-scoped fresh bounce budget contradicts the engine, whose budget is per-producer and history-derived, and the jump entry's shape is undecided although loom's coherence check rejects unknown outcomes
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D12
    summary: The invariant binds every refusal a run can reach, but landing refusals a loom run reaches are out of scope, so it is violated the day it lands
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Open risks, support-log.md ledger item 17
    summary: goto re-entry into loom's fresh-start coherence gate is deferred to the Plan, but it is a design decision the Discussion can and must make
  - id: F6
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D6, D8
    summary: A correctness-class parent-write consumed at one batch leaves Master's content in the tree for later batches, and the record takes no position
  - id: F7
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D10
    summary: rebaseline's refusal is keyed on cards, but batch records are keyed by batch number, so a plan edit that regroups begun batches passes unrefused
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md D5
    summary: A verify cannot run is undefined, so an I/O failure while re-running verify is claimed by both D5 (fail the batch) and D2's grouped transient row (halt, no mutation)
  - id: F9
    severity: LOW
    location: _lyx/discussion/decision-record.md D11
    summary: goto writes state running with the lock free and history non-empty, which loom's entry observation files as a crashed driver
  - id: F10
    severity: LOW
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: The per-row take-the-way-forward test assumes a refusal, but policy-class rows never halt, and the run progresses is undefined for webster verb rows
  - id: F11
    severity: NIT
    location: _lyx/discussion/decision-record.md Auto-mode assumptions
    summary: Several assumptions restate D4, D5, D11 and Scope rather than pointing at them, so a fix to D4 leaves a stale copy behind
---

### [BLOCKING] Run-exit audit cross-check not addressed; recover-batch named as an audit site it is not

**Location:** `decision-record.md` D5, D8 (also D6).

**Issue:** `internal/websterengine/runlevel.go` `runExitAuditCrossCheck` re-runs `CheckParent` and `CheckFork` over Master's whole session at run exit and hard-errors on any violation.
D5 records a policy finding as a warning and D6 dispositions a correctness finding by failing the batch, but neither touches this backstop,
so the same finding fails the run at exit.
The must-cover scenario "a `named-spawn` parent finding at batch 2 → batch 2 records with a warning" then ends in a failed run.
This is a false stop the record creates.
Separately, D5 ("when record-batch or recover-batch finds only policy-class violations") and D8 ("consumed by the first record-batch or recover-batch that reports it") treat recover-batch as an audit site.
`internal/websterengine/recoverbatch.go` calls neither `CheckFork` nor `CheckParent`; the only audit sites are record-batch and run exit.

**Fix:** Name run exit as the second audit site in place of recover-batch.
Decide that the run-exit cross-check applies the same classification and passes every finding already dispositioned in `state.json` (recorded warning, or consumed by a D6-failed batch);
it hard-errors only on an undispositioned finding.
Add a must-cover scenario for a run that exits after a recorded warning.

### [BLOCKING] D4's parent-write split overlaps and leaves run state as policy class

**Location:** `decision-record.md` D4 (restated in Auto-mode assumptions).

**Issue:** Policy class covers "a path outside the worktree or under a git-ignored path";
correctness covers "a tracked-tree path or a path in the reports directory".
In a hub worktree `_lyx` is a link into the weft, excluded from the warp through `.git/info/exclude`, and webster's reports dir is `_lyx/webster/reports` (`websterengine.ReportsDir`).
A Master write to a report therefore matches both rules, and D4 does not say which wins.
The same holds for `_lyx/webster/state.json` and `_lyx/plan/`, which are tracked in the weft but ignored by the warp:
"tracked tree" does not say which repo, and read as the warp they fall to policy class.
A Master write to `state.json` corrupts run state, which D2 defines as correctness, and D5 would downgrade it to a warning once verify passes.

**Fix:** Make correctness the rule that wins, and state it by location:
correctness is a path in the warp's tracked tree or anywhere under the run's `_lyx` content (reports, state, plan);
policy is everything else, meaning a path outside the worktree or a git-ignored warp path outside `_lyx`.

### [MEDIUM] goto's fresh bounce budget contradicts the engine, and the jump entry's shape is undecided

**Location:** `decision-record.md` D11.

**Issue:** D11 gives "a fresh bounce budget for that row's segment" and "appends a history entry recording the jump".
In `internal/shedengine`, the budget is per-producer and derived from `history[]`: `episodeStuckCount` counts a producer's Stuck entries back to its own latest `done` entry,
and `ProducerDef.Segment` "does not scope the bounce budget".
So "segment" names no existing budget, and an appended entry resets nothing unless the count learns to treat it as a boundary.
The entry's outcome is also undecided.
A new outcome value breaks `internal/loomengine/coherence.go`, which fails any history entry whose outcome is not `done`, `stuck` or `awaiting`.
A `done` entry would falsely record a producer verdict.
The Plan would have to invent a status-file contract change that a loom gate reads.

**Fix:** Decide the mechanism.
goto appends one entry with a new history-only outcome `goto`, naming the target producer.
`episodeStuckCount` treats it as an episode boundary for every producer sharing the target's `Segment` (the target alone when `Segment` is empty).
Loom's coherence outcome check accepts it.
State this in the Constraints so the change to the status-file history contract is visible.

### [MEDIUM] D12's invariant is violated the day it lands

**Location:** `decision-record.md` D12, against Scope Out.

**Issue:** "Every refusal a run can reach names its way forward … and has a row" is unscoped.
A loom run reaches landing refusals through its Finalize row, and batten runs gain `goto` and reach batten refusals.
Both are out of scope with no row, so the invariant is false when committed.
The "same commit" clause would also bind a new landing refusal to a table with no landing section.

**Fix:** Scope the invariant to the modules that have a section in `refusal-spec.md` (webster, shed and loom today).
State that another module joins when its audit adds its section.

### [MEDIUM] The goto re-entry gate is deferred when it should be decided

**Location:** `decision-record.md` Open risks (fourth bullet); `support-log.md` ledger item 17 and Notes for the plan writer.

**Issue:** The stencil says "Design the full scope now" and forbids "add this later" deferrals.
This question has one concrete instance: `loomengine.CheckSeed` runs only in the `Loom-Preflight` row (`internal/loomshed/loompreflight.go`).
Its fresh-start check fails `CheckHalfFinished` whenever history names a producer outside `Preflight`/`Loom-Preflight`, so a `goto` to either row on a run past them always halts there.
That is decidable now, and deferring it leaves the Plan to set goto's target policy.

**Fix:** Decide it.
goto places no restriction on targets, and an entry gate that refuses a re-entered run is an ordinary table row with its own way forward.
For `Loom-Preflight`'s half-finished check, the way forward is `goto` to the row after it, or seeding a new run.
Remove the open risk and close ledger item 17.

### [MEDIUM] A consumed correctness-class parent-write leaves unaudited content in the tree

**Location:** `decision-record.md` D6, D8.

**Issue:** A Master write into the tracked tree at batch N fails batch N (D6), and D8 marks the finding consumed.
The written file stays in the worktree, and D6's recovery "continues from the committed tree".
A later batch or the recovery strand can commit that content, and no audit reports it again.
The record takes no position on this.
The second Open risk covers a fork's bad commits, not Master's writes.

**Fix:** Take a position.
D6's failed-batch digest names each parent-write path, so the recovery strand sees the paths as suspect and reverts or re-derives them.
D8's single consumption stands.
Record the remaining exposure as an open risk.

### [MEDIUM] rebaseline's refusal misses regrouped batches

**Location:** `decision-record.md` D10.

**Issue:** `State.Batches` is keyed by batch number (`map[int]*BatchState`), and the batcher derives batch composition from the plan.
A plan edit can add or move a card so that a begun or terminal batch number now covers different cards, even though no card was removed or renumbered.
D10 refuses only removal or renumbering of a card, so rebaseline would accept that plan and attach old batch records to new card sets.

**Fix:** Refuse when any begun or terminal batch's card set under the edited plan differs from the one its record was begun with, with the same way forward.

### [LOW] "A verify cannot run" overlaps D2's grouped I/O row

**Location:** `decision-record.md` D5, against D2.

**Issue:** D5 sends "a verify cannot run" to D6, which mutates state: it archives the report and fails the batch.
D2 sends raw I/O failures to a transient row that halts without mutating.
A failed `exec`, a timeout, or an I/O error while reading a card's `verify:` line could be either.
The escalation cannot wedge, since D6 ends terminal with `recover-batch` as the way forward.
The split still needs a rule so the Plan can classify each failure.

**Fix:** Define it.
A verify command that fails to start, exits non-zero, or times out fails (D6).
A failure before any verify command starts, such as reading the plan or the cards, is D2's grouped transient row and mutates nothing.

### [LOW] goto's `running` write reads as a crashed driver

**Location:** `decision-record.md` D11.

**Issue:** `loomengine.DetectCrashResume` fires on state `running`, a free run lock, non-empty history and no step-handoff marker.
A `goto` leaves exactly that, so the next `lyx loom run` files a spurious crash-resume self-report issue.

**Fix:** goto writes state `paused` instead.
The next `step` or `run` resumes it through shedengine's existing resume write.

### [LOW] The per-row test criterion does not fit every class

**Location:** `decision-record.md` Acceptance criteria, third bullet.

**Issue:** The test "reaches the refusal … asserts the message names the way forward … takes that way forward".
A policy-class row never refuses; it warns and records.
"The run progresses" is also undefined for a webster verb, which has no shed run.
Transient rows are fine: releasing a held lock or running the named git command in the fixture is achievable.

**Fix:** Split the criterion.
Halting rows take the named way forward and assert that the refused verb, or the verb the way forward names, then succeeds.
Policy rows assert the warning is recorded and the verb succeeds.

### [NIT] Auto-mode assumptions restate decisions

**Location:** `decision-record.md` Auto-mode assumptions.

**Issue:** The `parent-write` split, the verify meaning, the out-of-scope list and the done-run rule copy D4, D5, Scope and D11 word for word.
The D4 copy goes stale as soon as D4 changes (F2).

**Fix:** Keep each assumption as the self-pick it records, with a pointer to the decision, and drop the restated content.

## Focus directive verdicts not raised as findings

- D11, how goto obtains the producer list: `shedverbs.Spec` already carries `Routing`, the told projection of the recipe's producer graph, so validating against it needs no path and no resolver.
  This is a Plan-level detail and not a defect; "segment" is covered by F3.
- Acceptance, transient rows: achievable (F10 covers the policy-row wording only).
- Notes for the plan writer: the identifiers there (`SeenForkTranscripts`, `ErrCardNotDone`) are names, not the signatures, `file:line` citations or dependency lists the stencil forbids, and the subsection is marked unverified.
  Not a defect.
- Rationale lines: short and plan-relevant; no deliberation narrative to relocate.
