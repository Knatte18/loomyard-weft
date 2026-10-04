---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: The fork point is defined as the pair's first weft commit, which is not the fork for an adopted or repaired pair, and the bound leaves task deletes, parent deletes and Publish untested
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 and D2
    summary: A whitespace-only PATTERN.md silently disables every directive and the D2 checker has no finding for an empty overview, so the real-file test passes on it
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9
    summary: D9 defers the durable-content move list and inbound links to the board entry, though they are requirements the plan writer needs
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D14
    summary: D14 says to see Open risks for the pre-aa67fd1d7 assumption, but Open risks has no such item
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Open risks and D2
    summary: D2 makes verify fail in every pair forked before PATTERN reaches the parent, and this is only a risk line with no Decision or bound
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D13
    summary: D13 restates a rejected alternative, which belongs only in the support log
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D13
    summary: Never pruning the portals anchor directory leaves it behind forever, and the Decision does not say what removes it or that this is accepted
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: D7 webster template-test move and D11 landingshed fixture rewrite have no acceptance criterion
---

### [MEDIUM] Fork-point definition and D6 bound

**Location:** `decision-record.md` D6, Acceptance criteria.
**Issue:** The fabric doc says the pair's first weft commit is the origin-record commit made by `Add`.
That commit holds the parent's PATTERN content unchanged, so for a normal `Add` the definition is sound.
The adopt path takes over an existing weft branch, and a legacy-worktree repair or dormant-weft creation makes no such commit, so "first weft commit" is not the fork point there.
The carry would then compare against content the task already changed, or refuse wrongly.
The bound also says the carry never deletes a parent file the task did not delete.
It does not say whether a task that deletes PATTERN paths propagates the delete, what a parent-side delete counts as, or what happens when the task and parent changed the paths identically.
Acceptance criteria test only the changed, unchanged and diverged cases.
Publish is excluded in the bound but not tested.
**Fix:** State that the fork point is read from the origin record's commit, and that a pair with no such commit carries nothing and reports why.
State the delete rules: a task delete of a PATTERN path is carried, a parent delete counts as a change, and identical changes are no conflict.
Add acceptance criteria for a task delete, a parent delete, identical changes and Publish carrying nothing.

### [MEDIUM] Empty overview disables every directive unnoticed

**Location:** `decision-record.md` D4, D2.
**Issue:** D4 turns a whitespace-only file into no directive, and says the D2 test guards format.
D2's checker lists no finding for an empty overview, so loomyard's own file could be emptied and the real-file test would pass while every agent loses the directive.
The Decision states what it lets through, but its named bound does not hold.
**Fix:** Add an empty-overview finding to the D2 checker, with a fixture and an acceptance criterion.

### [MEDIUM] Move list left on the board

**Location:** `decision-record.md` D9, Notes.
**Issue:** `Plan-Write` reads only this file.
D9 says durable content moves "per the board entry's list" and fixes the inbound links "listed on the board".
The destinations are requirements: which source content goes to which package doc, and which files carry inbound links.
Leaving them as a pointer to the board makes the plan depend on a read the plan writer is not guaranteed to do.
**Fix:** Extract the moves and the inbound-link file list into D9 as one compact list.
Keep the Notes pointer to the board entry.

### [LOW] Dangling cross-reference in D14

**Location:** `decision-record.md` D14.
**Issue:** The bound says "see Open risks", but the pre-`aa67fd1d7` assumption is under Auto-mode assumptions only.
**Fix:** Add the item to Open risks, so D14's pointer resolves.

### [LOW] D2 failure in forked pairs has no bound

**Location:** `decision-record.md` Open risks, D2.
**Issue:** The risk line says pairs forked before PATTERN reaches the parent fail the D2 test until they pull it in.
That is a guard that blocks verify across the board, and the record states no bound on how long or how to clear it.
**Fix:** Add to D2 a bound: the failure is by design, and the way forward is stated in the failure message (pull the parent's `_lyx/PATTERN.md` and `_lyx/pattern/`).

### [LOW] Rejected alternative inside a Decision

**Location:** `decision-record.md` D13.
**Issue:** "Rewriting setting values was rejected as a heuristic" re-litigates what was not chosen; the support log already holds it.
**Fix:** Delete the sentence from D13.

### [LOW] Portals anchor directory lifecycle

**Location:** `decision-record.md` D13.
**Issue:** With the prune removed, an empty `_portals/<anchor>` stays after the last pair goes, and no Decision says that is accepted or what clears it.
**Fix:** State that the empty directory is left in place and that only the hub workspace file's folder list is built from existing entries.

### [NIT] Acceptance gaps

**Location:** `decision-record.md` Acceptance criteria.
**Issue:** D7's webster `template_test.go` move and D11's `config_test.go` fixture rewrite have no criterion.
**Fix:** Add one line for each.

Checked and found sound: D5's judge wording is for judging findings and matches the judge template's never-open rule; D10's allowlist is a closed list with a reason line; the Scope, Decisions and Constraints sections agree otherwise; the support log's Interview and Question ledger agree with the Decisions, and it holds no requirement missing from the record.
