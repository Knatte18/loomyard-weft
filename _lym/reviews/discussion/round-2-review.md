---
verdict: BLOCKING
findings:
  - id: F22
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:128
    summary: "D4 says `lym loom parent-review answer <slug> --ask <n>` records the answer against the ask in the pull request record, but the existing verb (internal/loomcli/parentreview.go parentReviewAnswerVerb) writes only burlerengine's run-wide ask record through AnswerParentAsk, answers only that record's latest ask, and refuses one past the deadline every ask carries (internal/burlerengine/parentreview.go). A PR-Review ask numbered in the pull request record collides with or is refused against the run's Code-Review, Discussion-Review and judge asks there, so no satisfied answer reaches the pull request record and `pr-continue` refuses for good."
    fix: "In D4's answer bullet, mark the answer verb as changed and name its store for a PR-Review ask. Either the PR-Review asks live in burlerengine's ask record under their own tag and seed, with `parent-review ask` recording the head beside them, and `pr-continue` reading that record; or `parent-review answer` gains a PR-Review branch that writes the pull request record. Then state how the verb tells a PR-Review ask from a review or judge ask of the same run, and whether a PR-Review ask carries a deadline and what its expiry does. Add an acceptance bullet: under autonomy, with an earlier Code-Review or judge ask in the run, `parent-review answer <slug> --ask <n> --satisfied` for the newest PR-Review ask is recorded and `pr-continue` then ends the review."
  - id: F23
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:131
    summary: "D4's Bound says only a recorded satisfied answer at the current head ends the review, but only `pr-continue` checks it. The row ends the review on a standing Continue flag once the tree is clean and the head gated (internal/landingshed/prreview.go judge, sessionStopWanted). So a commit made after `pr-continue` is gated and then ended on, at a head no ask was sent at."
    fix: "In D4's `pr-continue` bullet and its Bound, state that under recorded autonomy the PR-Review row's judge ends the review on the continue flag only when the newest ask was answered satisfied at the head it judges (the gated head equal to that ask's head). Otherwise the row clears the continue flag and the review stays open. Add an acceptance bullet: under autonomy, a commit after a passing `pr-continue` leaves the review open until a new ask at the new head is answered satisfied."
  - id: F24
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:139
    summary: "D4 keeps the closed pull request's cleanup under the continue flag, with `pr-continue` checking no answer there. Under autonomy the non-interactive session's `pr-continue` therefore sets Continue, the row's judge records ReviewEnded on the closed pull request (internal/landingshed/prreview.go judge), and Finalize reopens and merges a closed pull request whose review ended (internal/landingshed/finalize_pr.go). A close made on GitHub is overridden with neither operator nor parent, which contradicts D4's own sentence that a closed pull request to reopen is a D5 class 4 severe halt."
    fix: "In D4's closed-pull-request bullet, state that under recorded autonomy the session cleans the tree but `pr-continue` on a closed pull request leaves ReviewEnded unset, or is refused with the class 4 halt as its way forward. The row then reaches its existing closed-without-merge halt, which the orch relays as a class 4 severe halt. Add an acceptance bullet: under autonomy, a pull request closed on GitHub halts PR-Review and is never reopened by Finalize."
  - id: F25
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F10
    location: _lym/discussion/decision-record.md:177
    summary: "D5 counts class 5 as entries at the row in the status history, but appendOrFold folds a halt into the previous entry when both are budget-exempt Stucks with equal Producer, Output, RoutedTo and GateAttempts, bumping Repeats (internal/shedengine/run.go appendOrFold, foldsInto). `lym loom resume` writes no history entry (internal/shedengine/goto.go RecordResumer). So halt, resume, the same halt leaves one entry, the entry count never reaches two, and the two-per-row Bound does not hold."
    fix: "In D5's class 5 paragraph and its Bound, state that each history entry at the row counts as one halt plus its `repeats`, since lym folds identical consecutive halts into one entry, citing shedengine.HistoryEntry's Repeats field. Extend the D5 Review acceptance bullet so role-autonomy.md counts an entry's repeats."
  - id: F26
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    carries: F6
    location: _lym/discussion/decision-record.md:117
    summary: "D4 records the run's autonomy when PR-Review starts its session, but the row rebuilds the spec and starts or resumes the session in ensureSession after every pause and every judging stop (internal/landingshed/prsession.go ensureSession, resumeSession; prreview.go holdWhileCovered, wake). Re-recording there lets a mid-review toggle take effect, against D4's toggle criterion. Also, a goto onto PR-Review resumes the recorded SessionID rather than starting a new session, and is admitted only on a paused, blocked or failed run (internal/shedengine/goto.go gotoAdmitted), so D4's goto takeover does not hold as stated."
    fix: "In D4, state that the autonomy is recorded once, when the pull request record has none, and that every later ensureSession builds the spec's Interactive from the recorded value. State what re-records it: a goto onto PR-Review clears the recorded autonomy and the session fields, as settleSendBack clears the session, so the re-entry starts a new session. Name that the operator stops the waiting run first (`lym loom pause` or the existing stop) before the goto."
  - id: F27
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F7
    location: _lym/discussion/decision-record.md:99
    summary: "D3's Bound stops the run only on a judge ask the parent leaves unanswered past judge_parent_timeout_min. An ask that ends not-sent, because the judge's own timeout leaves no room, or no-parent, because no parent orch resolves, passes the parent-agreement gate with the rulings as they stand (internal/driftengine/judgeparent.go). With judge_interactive forced off, those rulings stand with neither operator nor parent, against the Bound's claim that only the parent agrees to them."
    fix: "In D3's Bound, extend the autonomy stop to a judge ask that ends not-sent or no-parent: under autonomy each also stops the run at Drift-Verify as a halt naming `lym loom resume`. Or state explicitly that those two cases let the rulings stand and why that is accepted. Extend D3's acceptance bullet to the case chosen."
  - id: F28
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    carries: F16
    location: _lym/discussion/decision-record.md:254
    summary: "D9 puts the autonomy line on the presence line's path, orchengine.WithPresence, but that function returns the pointer unchanged unless the presence reads slack (internal/orchengine/prompt.go). Under a terminal presence no start, resume, reload or pending-resume pointer carries the autonomy line, which D9's acceptance bullet requires."
    fix: "In D9, say that the autonomy line is appended to the same pointers whatever the presence, beside the presence step and not inside it: on the start, resume and reload pointers and the watcher's pending resume. Keep the acceptance bullet, adding the terminal-presence case."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
    answer: "D4 now names `lym loom parent-review ask` (new), who sends each re-ask and where each ask's head is recorded. The answer side's store is a new defect, raised as F22."
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: open
    answer: "The recorded value is named, but which session start records it is not, and the goto takeover does not start a new session. Carried as F26."
  - id: F7
    status: open
    answer: "The timeout case is bounded; the not-sent and no-parent end states still let rulings stand. Carried as F27."
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: open
    answer: "The status history is an acceptable source, and the fixer's reason against decision entries holds. But a halt folds into the previous entry, so the count must include repeats. Carried as F25."
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: open
    answer: "The pointers are named, but the path named appends nothing outside a slack presence. Carried as F28."
  - id: F17
    status: resolved
  - id: F18
    status: outdated
    answer: "D9 no longer mails orchs on a write, so the bullet's mechanism is gone; the fixer's replacement bullet covers the call-time read."
  - id: F19
    status: resolved
  - id: F20
    status: resolved
  - id: F21
    status: resolved
---

### [BLOCKING] The PR-Review answer verb writes a different store than D4 says

**Location:** `_lym/discussion/decision-record.md:128`

**Issue:** D4 presents `lym loom parent-review answer` as recording into the pull request record.
`parentReviewAnswerVerb` writes only burlerengine's run-wide ask record, answers only that record's latest ask before its deadline, and shares its numbering with the run's review and judge asks.
As written, an autonomous review never records a satisfied answer where `pr-continue` looks for it.
All four lenses raised this.

### [BLOCKING] A standing continue flag ends the review at a head no ask covered

**Location:** `_lym/discussion/decision-record.md:131`

**Issue:** The row's `judge` ends the review on `Continue`, a clean tree and a gated head, without the satisfied-answer check D4 puts only in `pr-continue`.
A commit after `pr-continue` is gated and then ended on, so the parent's review is bypassed, and D1's Guard rests on that review.

### [BLOCKING] A closed pull request is reopened and merged with no one's word

**Location:** `_lym/discussion/decision-record.md:139`

**Issue:** Under autonomy the session's `pr-continue` on a closed pull request sets the continue flag, and the row records `ReviewEnded`.
Finalize then reopens and merges the pull request (`internal/landingshed/finalize_pr.go`), though D4 itself names a closed pull request a class 4 severe halt.

### [MEDIUM] Class 5 counts entries that lym folds together

**Location:** `_lym/discussion/decision-record.md:177`

**Issue:** Identical consecutive halts fold into one history entry with `repeats`, and a resume writes no entry, so an entry count never reaches two.

### [MEDIUM] Which session start records the autonomy, and the goto takeover

**Location:** `_lym/discussion/decision-record.md:117`

**Issue:** `ensureSession` rebuilds the spec on every resume within one review.
A goto resumes the recorded session rather than starting a new one, and is admitted only on a stopped run.

### [MEDIUM] Judge asks that never reach the parent still let rulings stand

**Location:** `_lym/discussion/decision-record.md:99`

**Issue:** The not-sent and no-parent end states pass the parent-agreement gate, and D3's autonomy stop covers only the timeout.

### [LOW] The presence path appends nothing outside a slack presence

**Location:** `_lym/discussion/decision-record.md:254`

**Issue:** `WithPresence` returns the pointer unchanged unless the presence is slack, so the autonomy line would be missing under a terminal presence.

## Focus verdicts

- D7 step 1: the trigger now matches `judgeOpenable`, which refuses on an open merge task or a conflict and passes a beam only behind its parent.
  A drain, merge-parent, drain loop repeats only on a new real conflict on the parent, never on lag alone, so no unbounded loop on a moving parent.
  Not a defect.
- D2: the halt carries the run's value, so the opt-out and the forced value both hold at whichever orch `orchroute.FromPair` reaches.
  Not a defect.
- D4: settled as F22, F23, F24 and F26.
  Each ask is sent by the row or by `parent-review ask`, and its head goes into the pull request record.
  The answer's store is wrong, and a re-gate is sound.
  A mid-review toggle is under-specified.
- D5: every halt has a route through class 6, and the class 5 count is F25.
- D3, D6, D8, D9: each introduced edge states a bound.
  The stores and readers the record names exist: the fork-point test row, `orchengine.RenderRoleFile`, `lym config beam --resolved` and the `.lym/beam/` marker directory.
  The gaps are F27 and F28.

## Rejected

- interactions F-I4 (an autonomous orch's replies on a non-autonomous run's halts lost to the hook): D8 has the orch open every reply meant for the operator with `[operator]` while the presence is slack, whatever its autonomy, so those replies pass the filter. Only a forgotten marker is lost, and D8 states that.
- The lenses' findings on D4's session recording, the judge's not-sent and no-parent asks and the class 5 fold are kept only as the carried F26, F27 and F25, and are not raised again as new findings.
