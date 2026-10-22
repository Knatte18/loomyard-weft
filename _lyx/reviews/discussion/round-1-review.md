---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 (way forward), reused by D4
    summary: The way forward names lyx fabric commit, which stages only the weft pathspec and never commits the parent code checkout's dirt the guard decided on, and a tackler session is role-guard refused from it anyway.
    fix: In D2, replace "for example with `lyx fabric commit` there" with a step that clears tracked dirt in the parent's code checkout, for example "commit or discard them in <checkout> with plain git; the dirt is the operator's", and record in D2 who performs it (the operator or orch, never a task session) and that a commit there reaches origin with Finalize's parent push. Cite the evidence in D2 - `lyx fabric commit` stages `ScopedPathspec` into the weft worktree (internal/fabriccli/weft_verbs.go commit verb, internal/fabricengine/commit.go Commit) and `sandboxedRoleRefusal` refuses it from task sessions (internal/fabriccli/roleguard.go). Keep D4 rendering through the same function so it inherits the change.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md Goal, D4 Rationale
    summary: The driver's hand-over at a blocked stop spawns a successor whose first Step re-calls a blocked Finalize with no goto, so an automatic re-step into the same refusal bypasses D4's guard and writes a second halt note.
    fix: Add to D4 (or a new Decision) how the hand-over route is handled at a Finalize dirty-parent halt - either the hand-over successor's first step at a blocked stop passes the recipe's goto guard for the current row before stepping, or the record states the route as an accepted Open risk with its bound. Cite internal/sheddriver/driverules.go decideDrive (hand-over before tackler, binaryEligible true for blocked), internal/sheddriver/drive.go run and handOver, and internal/shedengine/run.go Step re-calling current_producer on a blocked run. Add a matching acceptance criterion.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 (second target kind), Acceptance criteria 5 and 8
    summary: D4's premise that a Finalize halt's last history entry is the row that routed into Finalize is false, since a Finalize Stuck appends its own entry, so the PR-Gate arm guards no tackler route and only refuses deliberate orch gotos onto PR-Gate.
    fix: Drop D4's second target kind (rows whose OnDone routes to Finalize) and its rationale sentence, and narrow acceptance criteria 5 and 8 to Finalize alone; record the evidence in D4 - internal/shedengine/run.go appendHistory appends Producer def.Name on a Stuck with no on_stuck, and internal/sheddriver/tackler.go gotoRows adds the routed-from row only when it differs from CurrentProducer. Move the lens-review narrative about the routed-from row to the support log.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 Bound (merge-record admit) against D2
    summary: When a fabric merge record exists and the parent is dirty, Merge aggregates both reasons, so WorktreeDirty is true and D2 renders the paths with commit or discard them, the exact text D4's admit claims never reaches the operator.
    fix: In D2, state that Finalize's dirty arm renders the paths and way forward only when the guard error does not also carry the merge-in-progress reason, and otherwise surfaces the error's own text; correct D4's Bound to name that outcome. Add an acceptance criterion for a dirty parent with a fabric merge record.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: D4 calls its arm advisory, protecting no correctness, yet it refuses, while PATTERN-refusal-way-forward and the refusal spec's policy-guard class say such a guard warns and records and never halts; the record never places the refusal in a refusal-spec class.
    fix: Add to D4 the refusal-spec class of the arm and why - for example the transient class, a condition that clears by a named action, refused without mutating state and naming the retry - so the refusal is reconciled with PATTERN-refusal-way-forward; or, if it is a policy guard, change the arm to warn and record instead of refusing.
  - id: F6
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D2 Rationale; support-log.md Rejected alternatives
    summary: The claim that a bare lyx loom resume on a blocked Finalize spawns a tackler is false, since the hand-back closes the attempt and the spawned driver steps Finalize directly.
    fix: Replace D2's last rationale sentence with the true reason for the goto - it passes the goto guard and fails fast while the parent is still dirty - citing internal/sheddriver/drive.go resumeOpenAttempt and attemptClosed and internal/shedengine/run.go Step on a blocked run; correct the matching rejected alternative in the support log.
  - id: F7
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D2 (rendering)
    summary: shedengine collapses every line break in a stuck reason to one space, so a multi-line path list persists as one space-separated line and a path holding a space cannot be told apart from two paths.
    fix: In D2, name a separator that survives internal/shedengine/run.go stuckReason and oneLine, for example each path backtick-quoted and joined by comma and space, and add it to acceptance criterion 1.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Acceptance criteria 1 and 2
    summary: AC1 requires every dirty path in the reason while AC2 and D2 cap the list at 20 plus and N more; both cannot hold past 20 paths.
    fix: Reword AC1 to require every dirty path up to D2's cap, deferring to AC2 beyond it.
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D1
    summary: D1's premise that run records never reach the code checkout's git status is stated without the evidence it rests on.
    fix: Add to D1 the evidence - the code checkout's `_lyx` is a directory link made through internal/fslink by fabricengine (junction.go), and `/_lyx` sits in the code repository's .git/info/exclude, so no file under it is tracked there and the guard's tracked-only scope never lists one, on any platform.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Auto-mode assumptions bullet 2
    summary: That goto --to Finalize then resume steps Finalize on a run halted there is a fact about the code, settled by reading, yet it is recorded as an assumption.
    fix: Move the bullet into D2 as a settled fact with its evidence - internal/shedengine/goto.go gotoAdmitted admits the current row of a blocked run, Goto leaves it paused, and internal/loomcli/resume.go spawns a Go driver that steps it - and remove it from Auto-mode assumptions.
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Notes for the plan writer bullet 2
    summary: The note tells D4 to reuse halt.go's parent opening, which D3 deletes as parentDirtyPathsFromFabric.
    fix: Reword the note to name loomCLI.recordedParentBranch and fabricengine.OpenParent directly, without pointing at halt.go.
  - id: F12
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Scope, Out bullet 2
    summary: The sentence about the old halt-note section listing the wrong tree for task-side refusals is deliberation, not scope.
    fix: Move the sentence beginning "The old halt-note section listed the parent's paths" from Scope's Out to the support log's Turn 3 entry for the goal-scope finding; the Out bullet keeps only what stays unchanged.
---

### [BLOCKING] The way forward names a command that cannot clear the dirt

**Location:** `_lyx/discussion/decision-record.md` D2, reused by D4.
**Issue:** `lyx fabric commit` stages `ScopedPathspec` (`_lyx` plus the configured pathspec) into the weft worktree (`internal/fabriccli/weft_verbs.go` commit verb, line 124 for the pathspec, `internal/fabricengine/commit.go` `Commit`).
The guard decides only on the parent code checkout's tracked dirt (`internal/fabricengine/mergeguards.go` `pairDirtyReason`), such as `docs/overview.md`.
An operator following the example ends with the same dirt, and the next guarded goto refuses again.
The tackler, which reads the stuck reason, is a sandboxed task session that `sandboxedRoleRefusal` (`internal/fabriccli/roleguard.go`) refuses the verb from.
A commit of the parent's dirt also travels to origin with Finalize's parent push, which the record never states.

### [MEDIUM] Hand-over re-steps a blocked Finalize with no goto

**Location:** Goal; D4 Rationale.
**Issue:** At a blocked stop `binaryEligible` is true, and `decideDrive` picks hand-over before the tackler rule when the on-disk build changed (`internal/sheddriver/driverules.go`).
The successor finds no open attempt and calls `Step` (`internal/sheddriver/drive.go` `run`), and `shedengine` re-calls `current_producer` on a blocked run (`internal/shedengine/run.go`).
Finalize re-runs into the same refusal and a second halt note is written, which is the issue's third point.
Deploys after every landing make a changed build routine in this hub, and the record neither covers nor bounds the route.

### [MEDIUM] D4's routed-from premise is false

**Location:** D4; acceptance criteria 5 and 8.
**Issue:** A Finalize Stuck with no `on_stuck` appends Finalize's own history entry (`internal/shedengine/run.go` `appendHistory`, `Producer: def.Name`).
`gotoRows` adds the routed-from row only when it differs from the current row (`internal/sheddriver/tackler.go`), so at a Finalize halt the tackler can name only Finalize.
The PR-Gate arm therefore guards no tackler route, and it refuses orch gotos onto PR-Gate (loom and darn recipes) for parent dirt that does not concern PR-Gate.

### [MEDIUM] Merge-record admit still yields the discard text

**Location:** D4 Bound against D2.
**Issue:** `Merge` aggregates `mergeInProgressReason` and `pairDirtyReason` before refusing (`internal/fabricengine/merge.go`), and Finalize's dirty arm keys only on `WorktreeDirty()` (`internal/landingshed/finalize.go`).
With a merge record and dirt both present, D2 renders "commit or discard them", the text D4's admit says the operator never gets.

### [MEDIUM] D4's refusal has no refusal-spec class

**Location:** D4 Bound.
**Issue:** D4 calls the arm advisory, with Finalize's guard as the authority, so it protects no correctness.
`PATTERN-refusal-way-forward` says a guard in `internal/loom*` that does not protect correctness warns and records rather than halts, and `contracts/specs/refusal-spec.md` gives a policy guard "warn and record; never halt".
The arm may fit the transient class instead, but the record never says so, which leaves the conflict open.

### [LOW] False reason for the goto in the way forward

**Location:** D2 Rationale; support log's rejected alternatives.
**Issue:** After the tackler's refused resume the driver records a hand-back, which closes the attempt (`internal/sheddriver/drive.go` `attemptClosed`).
A later bare `lyx loom resume` spawns a Go driver that steps the blocked Finalize directly, without a tackler.
The goto is still worth keeping because it is guarded, but the recorded reason is wrong.

### [LOW] Path list collapses to one line

**Location:** D2 rendering.
**Issue:** `stuckReason` passes the reason through `oneLine`, which collapses every line break to a space (`internal/shedengine/run.go`).
A path holding a space, which `trackedDirtyPaths` keeps verbatim, then reads as two paths.

### [LOW] AC1 contradicts the cap

**Location:** acceptance criteria 1 and 2.
**Issue:** AC1 requires every dirty path, while AC2 caps the list at 20 plus `and N more`.

### [LOW] D1 premise lacks its evidence

**Location:** D1.
**Issue:** The record states that `_lyx` is a link into the records checkout and so never shows in the code checkout's status, but cites no source.
I checked that it holds: the hub's `loomyard/_lyx` is a symlink into `loomyard-weft/_lyx`, and `/_lyx` is in `.git/info/exclude`.
The evidence belongs in D1, so the plan writer does not re-derive it.

### [LOW] A settled fact is filed as an assumption

**Location:** Auto-mode assumptions, bullet 2.
**Issue:** Whether goto then resume steps Finalize is a question about the code, and reading settles it: `gotoAdmitted` admits the current row, and resume spawns a driver that steps the paused run.
The stencil requires such a fact to be recorded as settled, with its evidence.

### [NIT] Note points at a function D3 deletes

**Location:** Notes for the plan writer, bullet 2.
**Issue:** `parentDirtyPathsFromFabric` in `halt.go` is removed by D3, yet the note directs D4 to reuse its opening.

### [NIT] Deliberation in Scope

**Location:** Scope, Out, bullet 2.
**Issue:** The sentence justifying the loss for task-side halts is reasoning, not a scope boundary, and belongs in the support log.

## Rejected

- The friction reflection stencil's `parent fabric status:` instruction: it is conditional on the section being present, so it stays correct for older notes, and naming it would be call-site enumeration, which the rubric excludes.
- D2's stuck reason under `PATTERN-refusal-way-forward`: `internal/landingshed` is outside the entry's declared `internal/webster*`, `internal/shed*` and `internal/loom*` scope, so the record's constraint is stricter than required, not a violation.
- D4's admit on an unreadable parent: the bound is stated, and Finalize's own steps report the case.
