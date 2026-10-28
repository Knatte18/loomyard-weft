---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4 first paragraph and Acceptance criteria (driver bullet)
    summary: D4 hands JudgesFirst the stop's reason, which a stop built from a status leaves empty, so a restarted or rechecked driver sends the landing escalate to the tackler first
    fix: In D4, replace "the reason being the stop's reason, which the driver's stop already carries" with "the reason being the stop's Reason, or for a stop built from a status its Error, which carries the persisted halt text, as the driver's budget-exhausted row reads both". In Acceptance criteria, extend the driver bullet with "a driver that starts or re-checks at a halted escalate stop, built from the status, also sends it to the arbiter with no tackler call".
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4, sentence "A pawl run has no decision record to add to, so its arbiter hands an escalate on."
    summary: The pawl claim rests on an agent's choice, not on a mechanism, while the pawl arbiter's prompt still advertises lym loom decision add and a plain resume re-runs the fixer
    fix: Replace the pawl sentence in D4 with a settled fact and its evidence. Either (a) keep a pawl escalate a hand-back. D3's predicate stays false on a pawl run, so the escalate keeps LandingStopPrefix there, which matches the board entry's "pawl keeps its own full treatment". Add an acceptance criterion that a pawl fixer escalate halts with LandingStopPrefix and hands back. Or (b) state that on pawl `lym loom decision add` refuses with the no-record refusal (internal/loomcli/decision.go:653-654), that the arbiter stencil reads a refused repair verb as a hand-on, and that a resume with no decision only re-runs a fixer that re-escalates, bounded by the arbiter's spawn cap. Choose (a) unless the writer records why pawl needs the arbiter route.
  - id: F3
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 Bound paragraph
    summary: The bound names the spawn cap as what stops an escalate-resume loop, but does not say that the cap is the run's shared arbiter budget, which such a loop can spend before a later Webster or tackler stop needs it
    fix: In D4's Bound, after "by its per-run spawn cap, which stops an escalate-resume loop", add "an escalate gives its fixer attempt back, so the fixer caps never stop that loop, and the loop spends the run's one arbiter budget, after which every later arbiter-eligible stop hands back to the orch".
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 first paragraph
    summary: D3 requires the new prefix to differ only from LandingStopPrefix, but a prefix that opens with the decision halt prefix would be read as a hand-back and refused by the arbiter
    fix: In D3, change "distinct from `LandingStopPrefix`" to "distinct from `LandingStopPrefix` and from `shedadapters.DecisionHaltPrefix`, so neither `arbiterstop.HandsBack` nor its decision-halt check matches it".
---

### [BLOCKING] JudgesFirst reads an empty reason on a stop built from a status

**Location:** `_lym/discussion/decision-record.md` D4, and the acceptance criterion "The driver sends such a stop to the arbiter with no tackler call".

**Issue:** D4 says the driver's stop already carries the reason.
For a stop decoded from a step envelope it does, but `stopFromStatus` (`internal/sheddriver/driverules.go:109-125`) sets `Reason` only for a paused status and carries a blocked halt's text in `Error`.
The driver builds such stops at its start check, on the moved-run recheck and when it starts from a status (`internal/sheddriver/drive.go:629`, `776`, `781`, `871`).
`facts` calls the seam at `drive.go:713-714`, and the other prefix rows already read both fields (`budgetExhausted`, `driverules.go:254`).
If D4 is built as written, a driver that restarts at a halted escalate passes `""` to `JudgesFirst`, the stop meets the tackler first, and D4's "with no tackler call" and its acceptance criterion fail on a normal restart path.
This is a false premise in a Decision that a faithful plan would carry into code, so it is BLOCKING.
Four lenses (interactions, existing-paths, walkthrough, bound-breaker) and the handler found it independently; they are merged here.

### [MEDIUM] The pawl escalate claim is a prediction, not a settled mechanism

**Location:** `_lym/discussion/decision-record.md` D4, the pawl sentence.

**Issue:** The record leaves the pawl outcome to the arbiter agent's judgement and cites no evidence for it, which is a fact the design rests on, so the rubric counts it as a deferred research question.
The pawl arbiter's recipe description lists `lym loom decision add` as a repair verb (`internal/loomcli/arbiterprompt.go:11-14`, `22`).
That verb refuses a run with no record (`internal/loomcli/decision.go:653-654`).
Pawl's fixer is pointed at `loomengine.DiscussionDecisionRecord(l)` (`internal/loomcli/landingdeps.go:245`), which no pawl row writes.
The arbiter can also choose a plain resume at the halted row.
That re-runs a fixer whose escalate gives its attempt back (`internal/landingshed/fixerrun.go:256-257`), so the run loops until the spawn cap instead of handing on.
The board entry says pawl "keeps its own full treatment", so keeping pawl's escalate a hand-back is the lower-risk fix.
Walkthrough rated this MEDIUM and bound-breaker rated it LOW; it is kept at MEDIUM because the round's focus names this claim.

### [LOW] The escalate-resume loop spends the run's shared arbiter budget

**Location:** `_lym/discussion/decision-record.md` D4, Bound.

**Issue:** The spawn cap is counted per run from attempt records (`internal/shedarbiter/call.go:117`, `188-189`), and the arbiter shares it with Webster stops and tackler escalations.
An escalate turn spends no fixer attempt (`fixerrun.go:256-257`), so the fixer caps do not stop the loop.
Repeated escalates therefore drain the budget that a later stop relies on.
The Bound names the spawn cap correctly as what stops the loop, but not this cost.

### [LOW] The new prefix must also avoid the decision halt prefix

**Location:** `_lym/discussion/decision-record.md` D3.

**Issue:** `arbiterstop.HandsBack` is true for any reason that opens with `shedadapters.DecisionHaltPrefix` (`internal/arbiterstop/arbiterstop.go:62-63`).
A new prefix chosen to open with it would be handed back and refused by the arbiter, which undoes D4.
D3 constrains the prefix only against `LandingStopPrefix`.

## Rejected

- bound-breaker finding 2's claim that the record makes the fixer caps the loop's bound: D4 attributes stopping the loop to the spawn cap, and the fixer caps to the re-run fix; only the shared-budget point is kept, as F3, at LOW.
- bound-breaker finding 3: a duplicate of F2, merged there.
- existing-paths' note on `internal/testkit/envkit/envkit.go:153`: the rubric's `Do not flag` list excludes incomplete call-site enumeration, and the lens did not raise it.

Focus checks, with no finding beyond those above:

- D1's removed reroute states what now lands unreviewed and what bounds it.
  The lock-wait bound holds: `parentlock` has no lease, and waiters queue (bound-breaker).
- D3 and D4 agree on which reader sees which prefix: the envelope mark, the driver's `HandsBack` row, `arbiterstop` and batten's re-step all read `landingshed.HandsBack`.
  The one gap is F1.
- Relocation: no Decision or Constraint carries deliberation narrative that belongs in `support-log.md`.
  D4's last Bound sentence on the hand-over order is a constraint the plan writer needs.
- No Decision files a task or edits this task's own board entry, and D5's edits touch stencils, recipes, specs, docs and `PATTERN.md` only.
- Open risks `None.`: the wall-time clause in D1 is bounded by the lock's rules and hides no research question.
  The accepted darn overlap-note loss is a stated Constraint.
  F2 is the one unsettled fact.
- The declared `PATTERN-shed-stop-chain` change text in the record matches the round's declared text.
