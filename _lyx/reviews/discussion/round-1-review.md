---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:141
    summary: D8's rule that the orch writes an operator decision into a running task's board body cannot be carried out, since boardengine's run lock refuses any write that touches a field other than status on an entry a run holds, and D5's auto-landing bound leans on that rule
    fix: In D8, replace the board-decision rule with a split by holder. A decision on an entry no run holds is written into that entry's body through `lyx board`, as before. A decision on a task a run holds is recorded with `lyx loom decision add <slug> --by parent`, which appends to that run's decision record, the record D2 has the agent read at PR-Gate and D6's `show --record decision` serves to peers. State that the board's run lock is unchanged. Rewrite D8's Rationale to match, and change the last Open risks bullet so it no longer claims D8 answers part of `orch-split`'s open question through the board.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:67
    summary: The in-flight peer set is named three ways (D3 status is running with halted runs included, D6 a running status, D6 show refusing a slug that is not running), yet board run statuses are `<state> · <producer>` with states running, paused, blocked, failed and awaiting and no halted state, so a literal reading drops an awaiting peer, such as one waiting at PR-Gate
    fix: In D6, define the set once as every board task whose status is a run status (any `<state> · <producer>` value, as boardengine's run lock recognizes it), whatever the state, the caller's own task excluded. Use that same wording in D3's overlap check, in D6's `show` refusal (refuse a slug whose status is not a run status) and in D6's Bound. Extend the `lyx loom peers` acceptance criterion so it lists a peer in the awaiting or blocked state.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:107
    summary: D5's Bound overstates what limits an agent approval that lands and deploys, because Merge-Review passed includes merge_auto_accept (default true) settling a spent budget with open non-BLOCKING findings, and overlap check found nothing substantive is the agent's own judgment rather than an independent bound
    fix: "Rewrite D5's Bound to state: the agent runs the same check the orch's PR-Gate fork runs today, and code review stays with Webster-Review and Merge-Review as today, where Merge-Review may have auto-accepted a spent budget whose open risks the agent reads per D3; what an agent approval lets through that an orch approval would not is a conflict with an operator decision the agent cannot read, bounded by D8's decision-recording rule; `alignment_pr_approve: false` withdraws the landing. Keep the one-rework-per-run sentence."
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:61
    summary: D2's residual forgery is left to stencil limits without saying how it compares with today, when `lyx loom approve`, `lyx loom reject` and `lyx loom review approve` already have no caller guard and no settings deny, so any agent in the pair can approve directly today
    fix: In D2's Bound, after the forgery sentence, state that this residual is no wider than today's, since the approval verbs carry no caller guard and an agent in the pair can already run them, and that the per-spawn token and the attempt and head checks defend against a stale or guessed file only.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:81
    summary: D4 does not say what happens to the alignment attempt when the gate falls back on timeout or an invalid verdict, so a verdict file written after the fallback has no defined outcome, while D5 states the stop and the drop for PR-Align
    fix: Add to D4's fallback bullet that on any fallback Go stops a still-live agent through shuttle's stop and closes the attempt, so a verdict written after the fallback is never applied. Add the case to the parent-review timeout acceptance criterion.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:134
    summary: D7's Rationale says the keys mirror the arbiter's, but loom's ConfigSpec classifies `arbiter`, `arbiter_timeout_min` and every reviewer model key CategoryNone (open to the local layer), while D7 closes the `alignment` model, so the stated precedent is false and the departure is unexplained
    fix: Replace mirrors the arbiter's keys in D7's Rationale with the reason for the departure, that unlike the arbiter's and the reviewers' model keys, `alignment` picks the model whose approve lands on the parent with no later gate, which meets PATTERN-config-layers' Guard category. Keep the defaults' arbiter precedent if wanted, stated as defaults only.
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:193
    summary: Two Decision behaviors have no acceptance scenario, the closure of `alignment` and `alignment_pr_approve` to the local layer and `run_settings` that D7 gives as its bound, and D6's refusal of `lyx loom peers` with no board dir
    fix: Add one acceptance criterion that a local-layer file or a task's `run_settings` setting `alignment` or `alignment_pr_approve` is refused while `alignment_timeout_min` and `skip_alignment` are accepted, and extend the `lyx loom peers` criterion so that with no board dir the verb refuses, naming that no board exists.
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:139
    summary: D8 leaves the orch stencil's After landing line that says, while the hub's deploy key is empty, to run `update-plugins.sh` at the next wake after an approval, which no longer arrives when the agent approves
    fix: Add to D8 that the After landing section's empty-deploy line is reworded so it does not depend on an orch approval, for example to run `update-plugins.sh` on the landing notice or at the next wake after any landing.
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:287
    summary: Question ledger Q4 lists uncertainty as an escalation trigger, while D3 makes an escalation without a named question invalid
    fix: In Q4, replace or uncertainty with or a named question it cannot settle, an escalation without one being invalid.
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:114
    summary: D6 says the verb runs from a task pair or the prime, but from the prime there is no caller's run, so the own-task exclusion and the parent's commits since the run's merge base have no subject
    fix: In D6, either drop or the prime, or state that from the prime it lists every task holding a run status and omits the parent-commits part.
---

### [BLOCKING] D8's board-decision rule collides with the board run lock

**Location:** `_lyx/discussion/decision-record.md:141`

**Issue:** `internal/boardengine/doc.go`'s Run lock section says a held entry is the scope its run executes, and the write critical section refuses any write whose net change on it touches a field other than status, returning `ErrRunLocked`.
D8 tells the orch to write an operator decision "into the body of the board entry it concerns, a running task's included", and frames the freeze as stencil policy covering findings only.
It is enforced in Go, and Scope does not list a boardengine change.
The rule matters beyond D8: D5 lets an agent approval land and deploy with no orch turn, and the orch fork it replaces inherited the operator's settled decisions (orch stencil, Parent-review).
The cold agent sees a decision only if it is written somewhere it reads, so an unwritable rule leaves the auto-landing blind to decisions on running tasks.
`lyx loom decision add` already appends a parent decision to a run's decision record, which D2 and D6 already route to the agent.

### [MEDIUM] The in-flight peer set is named inconsistently and excludes awaiting peers

**Location:** `_lyx/discussion/decision-record.md:67`

**Issue:** D3 reads "a board task whose status is running, halted runs included", D6 reads "a board task with a running status", and D6's `show` refuses "a slug that is not running".
The board's run status is `<state> · <producer>` (`boardengine.RunStatus`), with states from `shedengine` of running, paused, blocked, failed and awaiting, and there is no halted state.
Read literally, `running` drops an awaiting peer, and a peer awaiting at PR-Gate is the one closest to landing on the shared parent.
The plan writer has to guess which set is meant.

### [MEDIUM] D5's Bound overstates what limits an agent landing

**Location:** `_lyx/discussion/decision-record.md:107`

**Issue:** The Bound says an agent approval "lets through only a run whose Webster, Merge-Review and verify gates passed and whose overlap check found nothing substantive".
Loom's template sets `merge_auto_accept: true`, which settles a spent Merge-Review budget as accepted unless a BLOCKING finding is open, so passed includes open MEDIUM and LOW findings carried over.
The overlap check is the agent's own judgment, the thing being bounded.
What changes from today is narrower and nameable: the orch fork already left code review to Webster and Merge-Review (orch stencil, PR-Gate), so the loss is the orch's context of operator decisions, which is F1's subject.

### [LOW] D2's residual forgery is not compared with today's exposure

**Location:** `_lyx/discussion/decision-record.md:61`

**Issue:** The support log's Interview records that `lyx loom approve` has no caller guard, and no settings deny names the approval verbs.
D2 leaves the same-pair forgery to stencil limits without saying it is no wider than what those agents can do today.
Without that sentence, a plan writer may build defenses for a residual the design already accepts.

### [LOW] D4 leaves a post-fallback verdict undefined

**Location:** `_lyx/discussion/decision-record.md:81`

**Issue:** D4 falls back to the delivery prompt on timeout, a dead agent or an invalid verdict, but does not say that the attempt is closed or a live agent stopped.
D2's Bound only requires the verdict to come after the spawn, which a late file meets.
D5 states the stop and the drop for `PR-Align`, so the two points diverge.

### [LOW] D7 cites a false precedent for closing the model key

**Location:** `_lyx/discussion/decision-record.md:134`

**Issue:** `internal/loomengine/config.go`'s `ConfigSpec` classifies `arbiter`, `arbiter_timeout_min`, `arbiter_max_spawns`, `review` and the other model keys `none`, and only gate-switching keys such as `parent_review_wait_min` and the `*_auto_accept` keys are guards.
Closing `alignment` is defensible, but the Rationale claims a precedent the code does not follow and gives no reason for the departure.

### [LOW] Two bounds and refusals lack acceptance scenarios

**Location:** `_lyx/discussion/decision-record.md:193`

**Issue:** D7's attack-surface bound is that `alignment` and `alignment_pr_approve` are closed to the local layer and `run_settings`, and no scenario covers it.
D6's no-board refusal of the verb itself is untested too: the existing criterion covers only the two alignment points falling back.

### [LOW] D8 misses the orch stencil's empty-deploy cue

**Location:** `_lyx/discussion/decision-record.md:139`

**Issue:** The orch stencil's After landing section says that while the deploy key is empty, the orch runs `update-plugins.sh` "at your next wake after an approval".
With agent approvals there is no such wake.
This hub sets `deploy`, so the gap shows only on a hub that leaves it empty.

### [NIT] Q4 contradicts D3 on uncertainty

**Location:** `_lyx/discussion/support-log.md:287`

**Issue:** Q4 records "or uncertainty" as an escalation trigger, and the fork finding on unbounded "unsure" escalation was later fixed in D3, so the ledger's recorded resolution no longer matches.

### [NIT] D6's prime form has no caller run

**Location:** `_lyx/discussion/decision-record.md:114`

**Issue:** The default output's own-task exclusion and parent-commits section are defined relative to the caller's run, and the prime has none.

## Rejected

- D5 and D7, routing around the gate: `skip_alignment` and `alignment_timeout_min` open to the local layer and `run_settings` only send a review to the orch, and the coherence check refuses a timeout at or above `parent_review_wait_min`, so neither weakens the gate. Not a defect.
- D2, stale or guessed verdicts: the per-spawn token, the after-spawn attempt check and the head and pull request check bound a stale file and a guessed path. Only the deliberate same-pair read remains, which is F4.
- D4 and D5 state machine: a parent verb during the agent, a held notify, restart attach, the once-per-run durable reject count and timeout below `parent_review_wait_min` do not contradict each other. `PR-Align`'s skip on a matching decision record does not clash with an escalation writing none: a restart reuses the same-head verdict, and a new head spawns afresh. The one undefined path is F5.
- D3 versus D6, overlap data: changed paths, peer decision records and plans, plus the read-only git D2 allows over the hub's shared store, where every pair's branch is visible, reach the shared-package, contradicting-decision and same-PATTERN-entry kinds. Only the set definition is wrong (F2).
- D1 seams: `parentreview.GateConfig` is built in `internal/loomcli/wiring.go` and passed through `shedrecipe`'s `Env.ParentReview`, so a seam closure wired in loomcli keeps the package's import set. `lyx loom` verbs run from a pair. The orch stencil has the Parent-review, PR-Gate and The board sections D8 names, and `CLAUDE.md` has the production paragraph D5 changes. No false premise.
- D8's handled-together rule: it is orch stencil policy, and the orch stencil is in Scope, so it belongs to this task.
- Leanness: Rationale lines are short, Auto-mode assumptions are one line each, and the open risk on a wrong approval points at D5 rather than restating it. No relocation is needed. Every fixed fork finding in the support log is reflected in the decision record.
