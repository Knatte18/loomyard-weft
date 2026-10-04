---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:111-112 (D10 fold and import labels)
    summary: D10 says a fold passes D3 validation and also that dropped labels are never refused, but adding GitHub labels can produce a duplicate or a second type label on a note, which D3 refuses
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:113,119 (D10 close as the retry path)
    summary: The only recovery for a failed close after import is `close`, which closes as not planned with a required reason, so an imported issue ends not planned and without the slug comment
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:107,117,122 (D10 sequencing)
    summary: The recorded-issue, slug and into checks are said to run before any write, but not under the board lock, so two concurrent imports can both pass them
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:96-98 (D9 issues)
    summary: D9 introduces a hand-written issues list that hides an issue from intake list and makes import a no-op, but states no bound and no validation of the numbers
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:113-115 (D10 close)
    summary: close on an issue that is already closed is not stated, and import of an issue with two GitHub type labels is refused with the wrong reason (no usable type label)
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:14,50
    summary: Interview item 6 and Q9 still describe a retry through re-running import on a recorded open issue, which the orchestrator decisions removed
---

### [MEDIUM] Fold label handling contradicts D3

**Location:** `decision-record.md` D10, the `into` bullets, against D3.
**Issue:** D10 says "the fold passes D3's validation like any write" and also that the issue's GitHub labels are added "only if the config lists them; the others are dropped and reported, never refused".
These can collide, and D10 does not say which wins.
A configured GitHub label the target already carries would be a duplicate, which D3 refuses.
A configured type label on an issue folded into a note that already has a type label gives a second type label, which D3 refuses.
A fold into an entry that already holds an orphaned label is refused by D3's full-entry validation, whatever the issue carries.
The bound on a fold (what it can add, and what happens on a clash) is therefore not stated.
**Fix:** State in D10 that a fold adds only labels the target does not already carry, and never adds a type label to a note, reporting those as dropped.
Say that the remaining D3 refusals (an orphaned label already on the target) still apply and name the way forward.

### [MEDIUM] The failed-close retry closes with the wrong reason

**Location:** `decision-record.md` D10, the closing paragraphs; Open risks; the acceptance criterion on a GitHub failure.
**Issue:** `import` comments the slug and closes as completed.
If that close fails, the issue is recorded, `import` is a no-op, and the only recovery is `close`.
`close` requires a reason and closes as not planned, so a successfully imported issue ends as "not planned", and the pointer comment naming the entry is whatever the operator types.
The acceptance criterion says a GitHub failure "reports the written entry and the retry command", but never says what that command is.
**Fix:** Decide the recovery.
Either make `close` take an optional `completed` outcome, or have `import`'s failure report the exact `close` payload to run, and accept the not-planned state explicitly in D10.
Name the command in the acceptance criterion.

### [LOW] Intake checks are not stated to run under the lock

**Location:** `decision-record.md` D10 Sequencing.
**Issue:** The decision says both slug and `into` checks run before any board write, and the board write runs under the lock.
It does not say the recorded-issue, slug and `into` checks run inside that lock, so two concurrent `import`s can both see the issue as unrecorded and create two notes.
**Fix:** State that the checks and the write share one lock acquisition, with the issue fetch outside it.

### [LOW] D9 `issues` is an unbounded hiding route

**Location:** `decision-record.md` D9.
**Issue:** A hand-written `issues` number makes `intake list` skip that issue and makes `import` a no-op, so it can hide an open inbox issue.
The recorded rule in D10 shows the effect but D9 states neither the bound nor whether a number must be a positive integer.
**Fix:** State in D9 that a number is a positive integer, is not checked against GitHub, and that the effect on `intake` is the accepted bound.

### [LOW] Two intake edge cases are unstated

**Location:** `decision-record.md` D10 `close` and `import`.
**Issue:** `close` on an already closed issue is neither refused nor a no-op by statement.
An issue labelled with two configured type labels is refused as having "no usable type label", which misdescribes the cause.
**Fix:** Say `close` of a closed issue is refused, naming its state, and that `import` names "more than one type label" separately.

### [LOW] Support log keeps the removed retry design

**Location:** `support-log.md` Interview item 6 and Q9.
**Issue:** Item 6 says "an idempotent retry for a failed close" and Q9 says "re-running `import` on a recorded open issue only closes it".
Orchestrator decisions 2 and 3 replaced this with a no-op `import` and a separate `close`.
**Fix:** Update both lines to the `close` recovery, and note the change under Review rounds.

## Checked and not a defect

- D10 `import` never overwrites an existing slug and refuses an `into` that names none, each before any write, with the way forward.
- A recorded issue, from several entries or a hand-written `issues` field, is a no-op that names every recording entry.
- D12's Decision and Rationale describe the same cost, and the refusal naming `board.yaml` and the label is stated.
- D6 and the acceptance criteria state the path-only `--label` behaviour and the scope of the "no entry carries it" check consistently.
