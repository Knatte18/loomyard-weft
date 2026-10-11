---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8 (and D7)
    summary: D8 includes the shared seat directives unchanged, but they open the chair-reviewer channel before acceptance and tell the chair to end its turn, contradicting D8's bound and D7's in-turn wait
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8
    summary: D8's bound leaves a kept, live reviewer free to write outside its review file and to convey new findings by reply, neither of which the byte comparison or anything else bounds
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: A required advisor that ends done with a failing must-pass seat gate gets no Accept and does not end the step, so the chair waits on the marker until its timeout
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D4, first bullet
    summary: D4 claims the failure rules are unchanged but maps a reviewer wait error to a nil error, while today's decideRound returns that wait error as an error
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Two Acceptance criteria trace to no Decision - the stale-marker removal before the round starts, which chair-first makes the guard against an early release, and Validate refusing a seat gate on the chair
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks, first bullet
    summary: Open risks says an llm-tier smoke or watched run must confirm the turn-start signal, a requirement no Acceptance criterion carries
  - id: F7
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md Open risks, last bullet
    summary: The fork-audit risk for KeepPane runs does not hold against the code, which audits forks before the KeepPane cleanup branch
---

### [BLOCKING] Shared seat directives contradict D7 and D8's channel bound

**Location:** `decision-record.md` D8 ("the shared seat directives are included unchanged"), D7.

**Issue:** D8 bounds the channel as opening "only after Go accepted the review, so the reviewer's judgment is fixed before the fixer speaks", and states the reviewer "sends the chair nothing unprompted".
It then includes `contracts/stencils/seat/seat-directive-chair.md` and `seat-directive-advisor.md` unchanged, which say the opposite:

- The advisor directive says "Write your output when your task is done or when the chair asks you to wrap up", and "A question you cannot answer goes to the chair".
  The first lets the chair cut the review short before acceptance, which is the chair influencing the review D8 says it cannot influence;
  the second has the reviewer message the chair unprompted.
- The chair directive says "Send an advisor nothing before its up line arrives; if you are ready before then, end your turn and the line arrives once you are idle".
  D8 says the marker stands in for the up line, and D7 says the chair waits inside its turn and takes no held turn end.
- The chair directive says "Read them as your inputs when they exist", which points the fixer at the review file as soon as it exists, before Go's acceptance.

Each seat prompt therefore carries two contradictory rule sets with no stated precedence, and the bound D8 claims rests on the stencil rules that lose.
Go enforces none of the channel's timing: the chair has the reviewer's strand name from `advisor_names` from its first turn.

**Fix:** Decide in D8 how the Burler seats override the shared directive's conflicting lines — either a Burler-specific directive variant, or Burler stencil text that names which shared lines do not apply and why — and state the bound as what remains stencil-enforced versus Go-enforced.
D7 should state that the chair never ends its turn to await an up line.

### [BLOCKING] D8's bound does not cover the kept reviewer's writes or message-borne findings

**Location:** `decision-record.md` D8.

**Issue:** Today the reviewer's strand is removed when its run reaches done: `burlerengine` leaves `KeepPane` zero-valued and `shuttleengine/wait.go`'s finish removes the strand of a done, non-`KeepPane` run.
Under the seat table every advisor is `KeepPane` (`seatengine/prompt.go`, `seatSpec`), so the reviewer stays a live agent with edit tools through the whole fix, and the chair's questions wake it.
D8's bound names only the review file, which the post-fix byte comparison checks.
It leaves unbounded:

- the reviewer editing any other file — a target file or the fixer-report — after acceptance, which lands in the fixer's commits with nothing to catch it;
- a reply that names a defect absent from the accepted review, which the fixer may act on as an unreviewed finding that the review file never records.

The rubric's attack-surface rule requires the Decision introducing the edge to state what it can let through and what bounds it; here the stated bound covers a subset of what the new live seat can do.

**Fix:** Extend D8's bound: state that a reply may only explain an accepted finding, and that the fixer neither acts on nor fixes anything a reply raises beyond the accepted review.
State what bounds the reviewer's writes after acceptance — a Go check, or an explicit stencil-only rule recorded with its residual risk under Open risks.

### [MEDIUM] A required advisor done with a failed gate is an uncovered path

**Location:** `decision-record.md` D3.

**Issue:** D3 calls `Accept` only when the required advisor "ends done with its gate passed", and ends the step only when it ends "in anything but done", its wait errors or `Accept` errors.
A required advisor that ends done with a failing must-pass seat gate (`shuttleengine`'s `GateOutcome.Passed` false) matches neither: no `Accept`, no marker, no step end, and the chair waits until the table timeout.
Burler's own reviewer entry lets the run through on cap (`burlerengine/doc.go`: "re-prompts at most reviewGateAttempts times and then lets the run through"), so Burler does not reach it today, but `Required` and the per-advisor gate are generic `seatengine` capabilities and the bound in D3 claims to cover every early end.

**Fix:** State in D3 that a required advisor that ends done with its gate failed ends the step as a non-done end does, or that `Table.Validate` refuses a must-pass gate entry on a required advisor; add the case to the Acceptance criteria.

### [MEDIUM] D4's wait-error rule changes today's behavior while claiming it is unchanged

**Location:** `decision-record.md` D4, first bullet; D4 title.

**Issue:** D4 is titled "Burler's failure rules map onto the seat result unchanged", yet its first bullet says "the reviewer ended non-done or its wait errored: the round returns the reviewer's outcome with a nil error".
Today's `decideRound` (`internal/burlerengine/handoff.go`) returns `burler: shuttle run: <wait error>` for a reviewer wait error, and a nil error only for a non-done reviewer outcome.
A wait error carries no terminal outcome, so "the reviewer's outcome" is empty there, and `BurlerProducer` would see an outcome-less, error-less round.
The Acceptance criteria list the died-or-timed-out case and omit the wait-error case.

**Fix:** Keep today's rule — a reviewer wait error is returned as an error — or record the change as its own decision with what `BurlerProducer` does with an empty outcome; add the wait-error case to the Acceptance criteria either way.

### [LOW] Two Acceptance criteria trace to no Decision

**Location:** `decision-record.md` Acceptance criteria, third and ninth bullets.

**Issue:** "a stale marker is removed before the round starts" appears in no Decision; D11 covers removal only on Resume.
Under D5 the chair starts first and its `lyx shuttle await` returns at once on an existing marker, so this removal, ordered before the chair's start, is now the guard that keeps a fresh round's fixer from acting before Go has accepted the review.
"`Table.Validate` refuses ... a seat gate on the chair" also traces to no Decision; D3 states the `Required` and `Accept` refusals only.

**Fix:** State in D5 that the stale marker is removed before the chair starts, and in D3 (or D2) that a seat gate on the chair is refused.

### [LOW] An Open risk carries a requirement no Acceptance criterion holds

**Location:** `decision-record.md` Open risks, first bullet.

**Issue:** The premise D10 rests on is stated with its bound (Auto-mode assumptions, and the consequence here: a false notice after the settle interval, never a dropped one).
But the bullet ends "The llm-tier smoke or a watched run must confirm the turn-start signal", a requirement the plan writer has no Acceptance criterion for.

**Fix:** Add an Acceptance criterion that an llm-tier smoke shows a SendMessage-started advisor turn reads as mid-turn, or reword the risk so it carries no "must".

### [NIT] The KeepPane fork-audit risk does not hold

**Location:** `decision-record.md` Open risks, last bullet.

**Issue:** The risk says the fork audit must still be attached to a kept advisor's done result.
`shuttleengine/wait.go`'s finish runs `AuditForks` for a done `ForkSubagents` run before, and independent of, the `KeepPane` cleanup branch, so the risk is already ruled out by the code.

**Fix:** Drop the bullet, or turn it into an Acceptance criterion row on the cluster round if the writer wants it pinned.

## Verdicts on the focus directive's questions

- **D3 and D10.**
  D3 covers an advisor whose wait errors or whose `Accept` errors: both end the step and stop the chair.
  The uncovered path is F3.
  D10's deferral is bounded: it delays, never drops, and an advisor that stays mid-turn defers at most to the table timeout; not a defect.
- **D5 and D11.**
  No start order or resume combination lets the fixer act before acceptance, provided the stale marker is removed before the chair starts (F5).
  D11's removal of a found marker when both seats are live matches today's `resume.go`; not a defect.
- **D8.**
  The bound does not hold: F1 and F2.
  A reviewer altering its review file is caught by the post-fix byte comparison, including an alteration made mid-fix, which ends the round on the review-changed error.
- **D9 and D10 premises.**
  Both premises are stated as assumptions with a bound, in Auto-mode assumptions and Open risks.
  Reading a kept advisor's session state after its wait has returned is supported: `shuttleengine/sessionread.go` folds the signals on demand, and a done `KeepPane` run keeps its run directory.
  Only F6 remains.
- **Scope, Constraints, Acceptance criteria.**
  Traces fail for two criteria (F5).
  Constraints and Auto-mode assumptions carry pointers to Decisions, not deliberation; no relocation is warranted.
- **support-log.md.**
  Every requirement in the Interview, Rejected alternatives and Question ledger also appears in `decision-record.md`: the deploy-window risk and the turn-start question sit under Open risks, the reviewer-kept-alive premise under D10's Rationale and Open risks.
  No finding.
