---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: decision-record.md D6 / A7
    summary: The persisted pre-fix head has no stated lifecycle; the gate closure resets per shuttle run, so a later run can overwrite it, and a passed gate never clears it
  - id: F2
    severity: MEDIUM
    class: design
    location: decision-record.md D5
    summary: The delete way forward leaves a run with no contract files and the record never says who regenerates them or how the run then ends
  - id: F3
    severity: LOW
    class: design
    location: decision-record.md D5 / D6
    summary: Transcript evidence ("Master wrote later", "the run's own paths") counts a recorded Write call, not a successful one, and the own-path set stays valid after another party edits the same path
  - id: F4
    severity: MEDIUM
    class: decision
    location: decision-record.md D3 / R1
    summary: D3's fallback is a different mechanism (a Bash deny) with no stated bound, and the choice between the two is left to Plan-time verification
  - id: F5
    severity: LOW
    class: consistency
    location: decision-record.md D7
    summary: D7 states the rule, then a non-exhaustive "at least" row and Go file list that belongs in the optional notes, not in the Decision
  - id: F6
    severity: LOW
    class: decision
    location: decision-record.md D8
    summary: The shed-driven command sequence is deferred to the Plan producer without the rule that decides it
  - id: F7
    severity: LOW
    class: design
    location: decision-record.md D10
    summary: The scan does not say how a stencil sentence that forbids a denied command differs from one that recommends it
  - id: F8
    severity: NIT
    class: consistency
    location: decision-record.md D3 Rationale
    summary: The untraceable-transcript sentence is deliberation narrative with no requirement, already held by the support log
---

### [MEDIUM] Pre-fix head lifecycle is unstated

**Location:** decision-record.md D6, A7, Acceptance criteria.
**Issue:** D6 keeps "the closure's per-shuttle-run attempt semantics"; A7 persists "HEAD at the gate's first failed evaluation, normally the last batch head".
The closure (`newVerifyGate`) starts each shuttle run with an empty pre-fix head, so each run has its own first failure.
The record does not say whether a later run overwrites the persisted value or keeps the first, whether a passed gate clears it, or what `reset --to pre-fix` does with a value left by an earlier gate.
"Normally" also leaves open the case where the value is not the last batch head.
An overwrite after a rejected fixer commit and a re-run would point `pre-fix` at a state that already contains the work the reset was meant to undo, and the ancestor refusal would not catch it.
**Fix:** State in D6 one rule for write, overwrite and clear, and drop "normally" from A7.

### [MEDIUM] Deleting the contract files leaves no way to finish the run

**Location:** decision-record.md D5.
**Issue:** The way forward for a path that is not yet cleared is to delete the contract files and re-run the verb, and absence clears the finding.
D5 says only that Master "has normally already rewritten them".
In the case where it has not, the files are gone, nothing is stated about who writes them again, and a run with no `outcome.yaml` is in flight, not finished.
Absence then clears the finding while the run lacks its terminal judgment.
**Fix:** State in D5 what the run does after the verb accepts on absent files (Master is re-entered to rewrite them, or the verb's way forward names the next step), and that absence clears the audit finding only, not the run's need for the files.

### [LOW] Transcript evidence is weaker than the claimed bound

**Location:** decision-record.md D5 bound, D6 bound, R4, R5.
**Issue:** D5 clears on a Write or Edit "recorded" in Master's transcript, and D6 exempts paths the transcripts "record a write to".
A tool-use entry whose result was an error or a denial is also in the transcript, so it can satisfy both rules without anything having been written.
For D6 the set is also not bound to the content: a path the run wrote and that was edited by someone else afterwards is exempt from the dirtiness refusal, which R4 does not cover (it covers only the safe direction).
**Fix:** State that only a Write/Edit with a successful result counts, in both D5 and D6, and add the other-party-edit case to R4 as a stated residual, or bound the exemption to paths whose last write is the run's own.

### [MEDIUM] D3 fallback carries no bound and the choice is left open

**Location:** decision-record.md D3, A4, R1.
**Issue:** D3's own Bound says the hook "denies nothing".
R1's fallback has the hook deny a Bash command, a different guard, with no statement of what it lets through (an interpreter reached through `xargs`, `bash -c` or a wrapper) and what bounds it, which the attack-surface rule requires of a Decision that adds a guard.
Which design ships depends on a Plan-time verification, so the plan writer has no decision to plan against.
**Fix:** Make the fallback a Decision (D3b) with its stated bound, and state the condition that selects it (hook input rewrite unsupported or run after the permission step), so the plan writer has a rule and not an open choice.

### [LOW] D7 enumeration belongs in the notes

**Location:** decision-record.md D7, Scope line 6.
**Issue:** The rule (every refusal, row and stencil sentence that says a reset names the verb) is complete in the first sentence and matches Scope.
The "at least" row list and the Go file list are non-exhaustive enumeration the compiler, the D10 scan and Plan-Write's quarry lookups own; as Decision text they read as a requirement that is both unbounded ("at least") and perishable.
They carry no requirement beyond the rule, so the completeness-before-leanness test passes for moving them.
**Fix:** Delete the lists from D7 and put a shortened pointer in "Notes for the plan writer".

### [LOW] D8 defers the sequence rule

**Location:** decision-record.md D8, Q14.
**Issue:** "The Plan producer settles the exact sequence for a shed-driven run (whether re-stepping the Webster row follows `run --fresh` or replaces it)" leaves a behavioral choice to Plan, and the example list ends in an open "…".
**Fix:** State the rule: the list has exactly one step that re-enters the run, the reset step precedes it, and for a shed-driven run that step is whichever the shed Webster row's own contract names; remove the "…" from the example.

### [LOW] D10 scan cannot tell a prohibition from a recommendation

**Location:** decision-record.md D10, Acceptance criteria.
**Issue:** The scan looks for denied command forms in specs, stencils, skills and string literals.
A stencil or refusal that forbids the command ("never `git push --force`") would match the same text as one that recommends it.
Nothing says how the scan separates the two, or that the PATTERN background file itself is outside the scanned surface.
**Fix:** State in D10 that the scan matches a denied form only where it is offered as a step or way forward, or name the exemption mechanism (a marked prohibition), and name what the planted-violation test pins.

### [NIT] Narrative in D3 Rationale

**Location:** decision-record.md D3 Rationale, last paragraph.
**Issue:** "The Master transcript from `board-model` ran on another machine and is not reachable here, so why Master reached for a script at all cannot be traced" carries no requirement for the plan writer; the support log's Interview and Q16 already hold it.
**Fix:** Delete the sentence from D3's Rationale.
