---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/loomshed/redfirstgate.go:288
    summary: "Go refusals and holds a run meets inside the closed span still name `lym loom decision add` as their only way forward, which the scope line now refuses, so the run is stranded; the red-first hold lifts only on that record, so a darn run sent back to Darn-Write after a Code-Review round, whose bug has no test seam or which fixes no bug, has no way out at all"
    fix: "Give each site a way forward that works once the scope is closed. (1) Red-first: in internal/loomcli/decision.go scopeClosedRefusal, admit an add call whose every entry title opens with gateslot.NoTestSeamTitle or gateslot.NoBugFixedTitle, since such an entry records why a check does not apply and changes no scope; add rows for both titles to TestDecisionCalls_RefusedOnceTheScopeCloses, and say so in the refusal comment, the add help and internal/loomcli/doc.go. (2) In the same commit, make the way forward of internal/loomshed/darnwrite.go:32 (darnStopWayForward), internal/shedadapters/webster.go:38 (planConflictWayForward), internal/websterengine/recordbatch.go:122, internal/loomcli/wiring.go:1639 (loomRerouteCapWayForward), internal/loomcli/parentreview.go:155 and :363 (parentReviewWayDecision and the answer help) and internal/landingshed/fixerrun.go:361 (the Parent-Align escalate) name the closed-scope route as well: once the task's scope is closed, file the call as a board note for a new task with `lym board upsert` and kind note, then accept or resume as the row allows. Each changed refusal keeps a test that asserts its message, per PATTERN-refusal-way-forward."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: "internal/loomcli/decision.go#scopeClosedAt"
    summary: "scopeClosedAt reads past-the-line only from round review files in the live Code-Review run directory, and archiveRunDir empties that directory on a new generation or an entryClear; a run whose second generation settles through approveWithoutReview (review.amend_max_bounces 0), then stops at Drift-Recheck and is sent back to Darn-Write, reads as open again and decision add is accepted"
    fix: "In scopeClosedAt, count the line reached when the Code-Review run directory holds a round review, a generation record or a settle record (both carried across archiveRunDir by CarryRecords), or when a timestamped sibling the shedadapters.ArchivedRunDirSegment rule maps to the same subdir exists beside it; keep the stat-error refusal for every other read failure."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: "internal/loomcli/decision.go#scopeClosedRefusal"
    summary: "The board entry keeps feedback from a person through the PR agent outside the line, but the closed span runs through PR-Review and refuses `--by operator` too, no PR agent exists (internal/landingshed/prreview.go:43, no agent takes part), and the change description's claim that operator feedback on the pull request needs no record entry is no exemption the code makes"
    fix: "Leave the code in place and defer this finding to the run's parent: it rules whether the exemption is built by admitting `--by operator` calls while the run stands at PR-Review in scopeClosedRefusal (with a TestDecisionCalls_RefusedOnceTheScopeCloses row, and orch-role.md and the summary saying so), or is moot because no PR agent exists, and records that ruling itself."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/role-autonomy.md:22
    summary: "Stencils and role files the diff left alone still tell the parent or an agent to record a decision inside the closed span, against the entry's rule that the orch procedure and the parent's prompt say the parent files a note instead"
    fix: "Add the closed-scope route (a board note for a new task with `lym board upsert` and kind note; at a Code-Review decision escalation the `lym loom circling continue --in-scope` ruling) at each of these, worded for a run at or past its first Code-Review round: contracts/orchrole/role-autonomy.md:19 and :22-23 (the drift does-not-match stop from Drift-Recheck); contracts/orchrole/role-landing-halts.md:22 (a landing stop at Parent-Align after a started round); contracts/orchrole/orch-role.md:196, :324 and :327; contracts/stencils/darn/darn-template-write.md:53, :57, :83, :91 and :117; contracts/stencils/sampler/sampler-template-agent.md:38; and contracts/stencils/loom/loom-rubric-code-review.md:119 and :125, whose deferral to the parent names recording or refusing the correction with decision add."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: "internal/loomcli/decision.go#scopeClosedAt"
    summary: "No test drives scopeClosedAt over a real run directory; every decision verb test fakes scopeClosed and TestInClosedScope takes roundStarted as a bool, so the summary's covered case of a run sent back after a started round, the missing-directory path and the stat-error path are unpinned"
    fix: "Add a tier0 table test of scopeClosedAt in internal/loomcli over a t.TempDir() reviews directory, with rows: no code directory at Darn-Write (open), code holding round-1-review.md at Darn-Write (closed), an empty code directory at Darn-Write (open, or closed once F2 lands with its carried record present), and an arras run (no line); export the reviews-dir seam through the test file if needed."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedrecipe/entries_bouncer.go:88
    summary: "Nothing tests that the scope_closed key reaches BouncerConfig.ScopeClosed or that the darn, tapestry and sampler Code-Bouncer rows set it, so dropping the key or its pass-through reverts the closed-scope escalation wording to the refused decision add with no failing test"
    fix: "Add scope_closed to the table of the recipe decision-switch test in internal/loomrecipe (decisionswitches_test.go) that pins escalate_decisions per shipped row, wanting true on the darn, tapestry and sampler Code-Bouncer rows, and add a row to entries_bouncer_test.go whose config sets scope_closed and asserts the built Bouncer's escalation names the closed-scope route."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/shedadapters/bouncerdecision.go#answerRoute"
    summary: "At or past the hard cap on a closed-scope segment, the brief tells the parent to pass ids to --in-scope on the continue verb below, but decideVerbs lists accept alone there and circlingVerb refuses continue at the cap"
    fix: "Give answerRoute the round and, when b.atHardCap(round) holds, drop the --in-scope sentence so the closed route says only to file every call as a board note and then record the outcome with accept, matching noticeWayForward's at-cap case; add the at-cap closed-scope case to TestBouncer_DecisionEscalation's hard-cap subtest."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/prompt.go:557
    summary: "Branches the diff adds have no test row: writePriorFinding's inScopeMark case, the closed-scope hard-cap wording of noticeWayForward and escalationReason, RecordInScopeContinue's ErrCirclingDecided, round-0 and unreadable-review paths, InScopeFindings on a malformed file, and the circling preRun record closure that routes --in-scope to RecordInScopeContinue"
    fix: "Add a prompt test row with PriorFinding{Held true, InScope true} wanting inScopeMark and not heldByParentMark; add a ScopeClosed case to the hard-cap subtest of TestBouncer_DecisionEscalation; add rows to TestRecordInScopeContinue for a pre-existing decision file (ErrCirclingDecided, file unchanged), an empty run directory (ErrNotEscalated) and a missing review; add a malformed-file row for InScopeFindings."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/shed-recipe-spec.md:117
    summary: "The shed recipe spec documents each optional boolean Bouncer key (escalate_decisions, blocking_escalates, decisions_escalated) and which shipped rows set it, but not the new scope_closed key three shipped recipes now set"
    fix: "Add a paragraph after the blocking_escalates paragraph: a Bouncer row's optional boolean scope_closed key defaults to false; when true, its decision escalation's brief, notice and reason name the --in-scope ruling and a board note in place of a decision-record entry; the shipped Code-Review rows of tapestry, darn and sampler set it."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/decision.go:2041
    summary: "To fit the new refusal paragraph under the help cap, the decision add help drops contract facts the change leaves in force: a failed board copy is record_store_error and never fails the verb, --by and --amends-plan apply to every --from entry, all entries land in one commit or none does, and --drops-create lets Webster's done-check skip the target and is recorded as its own call"
    fix: "Restore those four facts in the add help's Long text, shortening the new scope paragraph or another paragraph instead, so the help stays under its length cap."
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/specs/loom-plan-spec.md:112
    summary: "The plan spec asks every card that creates or widens a tier0 test to state its seconds, unconditionally, while the plan-cards block and the plan-review rubric ask for the estimate only when the tiers hold a budget line, which a budget of 0 omits"
    fix: "Open the plan-spec rule with the same condition as the stencils: when the tiers the planner is given hold a plan tier0 budget line, a card that creates or widens a tier0 test states in its Intent the seconds the test adds."
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: internal/gateslot/config.go:212
    summary: "plan_tier0_budget_sec is classified CategoryCrossPair, but its value sets a check of the plan review's prose and 0 switches that check off, which PATTERN-config-layers names a Guard key"
    fix: "Classify plan_tier0_budget_sec as configengine.CategoryGuard in ConfigSpec's categories."
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:75
    summary: "The merge-danger blast radius names loom alone, but the diff also adds a gate.yaml key that a binary without it refuses on load once a hub sets it, and changes the Bouncer escalation and burler round check in shedadapters and burlerengine"
    fix: "Name the blast radius as loom, gate and the review perches (shed Bouncer and burler rounds), and add one sentence under Merge danger that an older binary refuses a gate.yaml carrying plan_tier0_budget_sec until the key is unset."
---

### [BLOCKING] Ways forward in the closed span name a verb the line refuses

**Location:** `internal/loomshed/redfirstgate.go:288`, with `darnwrite.go:32`, `webster.go:38`, `recordbatch.go:122`, `wiring.go:1639`, `parentreview.go:155` and `:363`, `fixerrun.go:361`

**Issue:** The entry asks that a refusal name the way forward and that the parent and agents file a note instead of trying.
`scopeClosedAt` keeps the scope closed for a run sent back to its writer after a Code-Review round:
the summary's Drift-Recheck goto case for darn, and Drift-Recheck's `goto --to Webster` for tapestry, which then runs Webster and Parent-Align again.
Every refusal or hold those rows can return still names `lym loom decision add` as its only way forward, and that verb now refuses.
The red-first hold is the sharpest case:
it lifts only on a new failing test or on the parent's `No test seam` or `No bug fixed` record.
A darn bug with no reachable seam therefore cannot move once the scope is closed.
The parent-review answer's way forward meets the same refusal at every Code-Review ask.
This breaks PATTERN-refusal-way-forward: a way forward that cannot be taken is no way forward.
The focus asked whether each row that can still send a run back has a way forward once both verbs refuse;
for these rows it has none.

### [MEDIUM] An archived Code-Review run directory reopens the scope

**Location:** `internal/loomcli/decision.go#scopeClosedAt`

**Issue:** `roundStarted` is `LatestRound(code) > 0`.
`startNextGeneration` and the `entryClear` case both call `archiveRunDir`, which moves the rounds to a `code-<stamp>` sibling and recreates `code` with only the carried records.
Take an amendment generation settled through `approveWithoutReview`, with `review.amend_max_bounces` at 0.
It writes no round, so a later Drift-Recheck stop and `goto --to Darn-Write` reads as before the line,
and `decision add` and `replace` pass.
This contradicts the commit message's claim that a run past its first round stays closed.

### [MEDIUM] The PR-agent exemption is not built

**Location:** `internal/loomcli/decision.go#scopeClosedRefusal`

**Issue:** The amended entry keeps feedback from a person through the PR agent outside the line.
`InClosedScope` holds PR-Review inside the closed span, and the refusal covers `--by operator`, which `TestDecisionCalls_RefusedOnceTheScopeCloses` pins.
No PR agent exists: `prreview.go:43` says no agent takes part.
The summary's "Feedback the operator gives on the pull request needs no entry in the record" and orch-role's "stays outside the line" assert the exemption, but nothing builds it.
Whether to admit the operator at PR-Review, or to rule the exemption moot, is the parent's call.

### [MEDIUM] Role files and stencils still send the parent to decision add inside the closed span

**Location:** `contracts/orchrole/role-autonomy.md:22`

**Issue:** The entry says the orch procedure and the parent's prompt state the line.
`orch-role.md` and the parent directive do.
The on-demand role files do not:
`role-autonomy.md` still settles a Drift-Recheck does-not-match stop "with a decision and a goto",
and `role-landing-halts.md` still records a Parent-Align landing stop with `decision add`.
`orch-role.md:196`, `:324` and `:327` are also unchanged.
On the agent side, the darn writer's stop, correction, drop and red-first asks and the sampler agent's operator-decision line are unchanged too.
The Code-Review rubric's parent-deferral bullets still name `decision add` as the parent's only sound change, and that rubric runs only inside the closed span.

### [MEDIUM] scopeClosedAt has no test over a real run directory

**Location:** `internal/loomcli/decision.go#scopeClosedAt`

**Issue:** The decision verb tests inject `scopeClosed`, and `TestInClosedScope` takes `roundStarted` as a given bool.
So the run-directory read that decides the sent-back case, the missing-directory path and the stat-error path fail no test when broken.

### [MEDIUM] The scope_closed recipe key and its pass-through are untested

**Location:** `internal/shedrecipe/entries_bouncer.go:88`

**Issue:** The decision-switch test pins `escalate_decisions` per shipped row but not `scope_closed`.
No `entries_bouncer` row sets it either.
Dropping it from a recipe, or dropping the `ScopeClosed: scopeClosed` line, would silently bring back the refused `decision add` route in every closed-scope escalation.

### [LOW] The closed-scope answer route names --in-scope at the hard cap

**Location:** `internal/shedadapters/bouncerdecision.go#answerRoute`

**Issue:** `answerRoute` ignores the round, while `decideVerbs` drops the continue line at the hard cap.
`noticeWayForward` already words the at-cap case without `--in-scope`, so the brief and the notice disagree there.

### [LOW] Added branches without a test row

**Location:** `internal/burlerengine/prompt.go:557`

**Issue:** These branches can each break with no test failing:
- the in-scope mark in the reviewer prompt;
- the hard-cap closed-scope notice and reason;
- `RecordInScopeContinue`'s documented `ErrCirclingDecided` contract and its round-0 and missing-review paths;
- `InScopeFindings` on a malformed file.

### [LOW] The shed recipe spec omits scope_closed

**Location:** `contracts/specs/shed-recipe-spec.md:117`

**Issue:** The spec lists every other boolean Bouncer and BurlerFix key and the rows that set it, but not the new key.

### [LOW] The decision add help drops facts still in force

**Location:** `internal/loomcli/decision.go:2041`

**Issue:** Commit 5102cf8a6 shortened the `--from`, `--drops-create` and board-copy text to fit the cap.
The facts it dropped about all-or-none commits, per-entry flags, `record_store_error` and Webster's done-check still hold.

### [LOW] The plan spec and the stencils disagree on when a card states its tier0 seconds

**Location:** `contracts/specs/loom-plan-spec.md:112`

**Issue:** The spec rule is unconditional.
`loom-block-plan-cards.md` and `loom-rubric-plan-review.md` ask only when the tiers hold a budget line, and a budget of 0 renders none.

### [LOW] plan_tier0_budget_sec is classified as cross-pair rather than guard

**Location:** `internal/gateslot/config.go:212`

**Issue:** PATTERN-config-layers's Guard category covers a key that switches off or hollows a review check and the prose a reviewer is given.
This key sets the plan review's budget check, and 0 turns that check off.
Both categories close the key to the local layer, so only the classification is wrong.

### [LOW] The merge-danger blast radius is narrower than the diff

**Location:** `_lym/landing/summary.md:75`

**Issue:** The blast radius names loom alone.
The diff also adds a gate config key, which an older binary refuses once a hub sets it.
It also changes the shed Bouncer escalation and the burler round check that every review perch runs.

## Rejected

- code-scope F1, the Code-Burler-Fix decision notice at `internal/shedadapters/burlerfix.go:105`: every shipped Code-Burler-Fix row sets `decisions_escalated: true` (darn :256, tapestry :475, sampler :213), so the row sends no notice of its own and the text is unreachable in the closed span.
