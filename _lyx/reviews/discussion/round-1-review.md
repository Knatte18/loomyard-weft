---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D8 and D10
    summary: decision add states what it never does but not what it lets through; --by is an unverified label and Plan-Write may append to its own sole input
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D7 and D4
    summary: accept on a budget escalation replaces a hard block with a parent-settleable pass and states no bound on what it lets through
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D4, D5 and D7
    summary: a CIRCLING ruling at a spent budget has no stated cause; a continue on it would hit Shed's generic block, the halt the goal removes
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D2
    summary: the guard accepts any open key spanning two rounds, but the definition of CIRCLING requires a gating finding on that key
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D2
    summary: the record does not say who computes the per-key rounds list the guard reads, Go or the judge
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D1 and D4
    summary: D1 says CONTINUE never extends past the review budget, D4 grants one exempt round past it
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D5 and D6
    summary: parent_notice is bounded as advisory but not as to its content, and the parent session acts on it as an instruction
---

### [MEDIUM] decision add has no stated bound on what it lets through

**Location:** `decision-record.md` D8, D10.
**Issue:** D8's Bound lists what the verb never does (edit, route, touch the support log).
It does not say what it now lets through.
It checks no caller identity, and `--by parent|operator` is a self-declared label.
D10 puts the verb into the Plan-Write stencil, so the agent that reads the record as its sole input can append to it and then plan on its own addition.
The rubric's attack-surface question requires the bound to be stated.
**Fix:** state in D8 that any shell addressing the run can append, that `--by` is an unverified label, and what bounds it: the heading marks every entry as added after Discussion, the append is a committed diff, and plan review and PR review read marked entries as additions to be checked.

### [MEDIUM] accept on a budget escalation has no stated bound

**Location:** `decision-record.md` D7, D4.
**Issue:** Before this change, a spent budget blocked the run for a human.
Now a parent fork's `accept` lets the run proceed past a segment that still has open gating findings, on a round whose verdict was plain `CONTINUE`.
D4 bounds the exemption only; D7 states no bound for `accept`.
**Fix:** add a Bound to D7: `accept` passes only the one segment at the one awaiting Bouncer row, once per round, recorded as unconverged in the committed decision file; Shed's budget block stays the backstop elsewhere.

### [MEDIUM] Circling at a spent budget has no stated cause

**Location:** `decision-record.md` D4, D5, D7.
**Issue:** D4 exempts a `continue` only on a budget escalation.
If the judge rules `CIRCLING` when the count has already reached the budget, the cause is unstated.
A `continue` recorded as circling is not exempt, so its `Stuck` is counted and Shed blocks with `bounce-budget-exhausted`.
That is the human halt the goal removes.
**Fix:** decide that the cause is `budget` whenever the count has reached `review_max_bounces`, and `circling` otherwise; add an acceptance criterion.

### [MEDIUM] Guard is weaker than the definition of CIRCLING

**Location:** `decision-record.md` D2.
**Issue:** The definition requires a gating finding on a key open in the judged round and an earlier one.
The guard requires only an open key spanning those rounds, so a non-gating recurring key lets `CIRCLING` through.
The acceptance criteria repeat the weaker wording.
**Fix:** the guard and the criterion require a gating finding on such a key.

### [MEDIUM] Source of the per-key rounds list is open

**Location:** `decision-record.md` D2.
**Issue:** D2 has Go render `rounds` into the facts file, but the guard reads "the ledger the judge wrote".
If the judge writes `rounds`, the guard trusts a value the judge can get wrong.
**Fix:** decide that Go computes `rounds` from the ledgers of the earlier rounds; the judge's ledger supplies only keys, status, class and severity for the judged round.

### [LOW] D1 and D4 disagree on the budget limit

**Location:** `decision-record.md` D1.
**Issue:** D1 says `CONTINUE` extends the loop "never past" the budget; D4 grants one exempt round past it on a recorded `continue`.
**Fix:** D1 says the judge's `CONTINUE` never extends past the budget by itself, and only a recorded decision (D4) grants one more round.

### [LOW] parent_notice content is not bounded

**Location:** `decision-record.md` D5, D6.
**Issue:** The bound says Go never reads the notice back.
It does not say what the notice may contain, yet the driver relays it verbatim and the parent session acts on it.
Text derived from judge or finding output in it would carry untrusted content into the parent as an instruction.
**Fix:** state in D5 that the notice interpolates only the slug and the brief path.

## Verdicts on the focus questions without a finding

- D12 settles the fix: run the tier, fix the tests if stale, production if it regressed.
  A plan writer can act on it; not a defect.
- Relocation: no Decision, Open risk or Auto-mode bullet carries narrative whose removal is needed, and the support log holds no requirement missing from the record.
