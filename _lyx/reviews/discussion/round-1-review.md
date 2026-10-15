---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D1 and Acceptance criteria
    summary: D1 excludes a done sibling Y but sets no status condition on the written task X, so a pair with a done X and an open Y is a sibling from X's side only, and the batch report-once rule leaves its outcome open
    fix: In decision-record.md D1, extend the first bullet to read that Y is not X, X's status after the write is not done, and Y's status is not done, and add a sentence beside the labels sentence that X's status is the one the write leaves on it. In Acceptance criteria, add X written with status done to the no-warning list.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D1
    summary: D1 treats every status other than done as open, so an abandoned Y waiting on D is a sibling, and neither the rule nor its rationale says whether that is intended
    fix: In decision-record.md D1, state whether a Y whose status is abandoned counts; the recommended pick is to exclude it, extending the status bullet to Y's status is neither done nor abandoned, with a rationale clause that an abandoned task is never started by a newly-ready start. Add the matching row to Acceptance criteria (no warning when Y is abandoned, or a warning if the pick keeps it), and add the pick to Auto-mode assumptions and the support-log Question ledger.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3
    summary: D3's line names every shared dependency once, but does not say whether that means every dependency X and Y both hold after the write or only the added ones that made them siblings
    fix: In decision-record.md D3, replace every shared dependency once with every dependency that makes the pair siblings under D1, once, so a dependency X and Y already shared before the write is not named.
---

### [MEDIUM] D1 sets no status condition on the written task X

**Location:** `_lyx/discussion/decision-record.md` D1 and Acceptance criteria.

**Issue:** D1's first bullet excludes a `done` Y, and its rationale says "A done task waits on nothing", which applies to X as much as to Y.
Nothing stops a write from leaving X `done` while adding to its `depends_on`: `UpsertTask` takes `status` and `depends_on` in one payload, and `validateWrite` in `internal/boardengine/store.go` refuses neither, and `merge`'s `set_status` can set X's status in the same write.
In an `upsert-batch` where X is written `done` and Y open, both newly on D, the pair is a sibling with X as the written task and not one with Y as the written task, since X as the candidate sibling is `done`.
D3's "reported once" rule assumes the relation is symmetric, so this pair's outcome is open.
The focus asked for the written-X-is-`done` case; the record does not settle it.

### [LOW] D1 is silent on an abandoned sibling

**Location:** `_lyx/discussion/decision-record.md` D1.

**Issue:** `internal/boardengine/doc.go` names `abandoned` as a status, and `Prune` keeps an abandoned entry.
An abandoned task is never started by the newly-ready start: `readySlugs` in `internal/battencli/land.go` takes only tasks with a nil status.
D1 keeps running siblings with a stated reason but says nothing about abandoned ones, so an abandoned Y waiting on D warns by default with no recorded pick.
Either choice can be argued; the record has to make it.

### [LOW] D3 leaves "every shared dependency" open

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** When X and Y already both wait on E and the write adds D to X, the pair is a sibling through D only.
D3's line names "every shared dependency once", which reads as naming E as well, while the Goal and the first Acceptance criterion name only the dependency the write added.
The plan writer has to guess which set the line carries.

## Focus directives judged with no finding

- D1 against the merge baseline and the no-warning list: they agree case by case apart from F1. A batch where Y is newly written is settled by D2's post-write board and D3's report-once rule.
- D3 and D2 bound: D3 states the warning changes no exit code, no refusal and no stored record, and Scope Out says it never blocks a write. The shared-lock read adds no edge or guard and needs no bound of its own. What the warning misses is in Open risks.
- Scope and D4: the reach to `upsert-batch` and `merge` is tied to the entry's goal in D4's rationale. `promote` changes only the kind of a note, which `validateWrite` keeps free of `depends_on`, and `intake` in `internal/boardengine/intake.go` writes no `depends_on`. D4's claim holds.
- Auto-mode assumptions and Constraints: the assumptions are pointers to the picks, which is that section's purpose. No Constraint carries a requirement missing from the Decisions.
- support-log.md Turn 3 and Question ledger: each fixed disposition matches the record. The rejected `set-deps` way-forward disposition holds, since `lyx board set-deps` is outside `PATTERN-no-denied-recovery`'s deny list.
