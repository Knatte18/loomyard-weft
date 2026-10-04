---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10 (import)
    summary: import creating a note states no bound on a slug that an entry already uses, nor on an unknown into slug, so an upsert-style create could overwrite an existing entry
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10 Bound (retry)
    summary: the retry of a recorded open issue is unbounded for an issue recorded by hand through D9, for an issue several entries record, and for other payload keys such as into
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D12 Rationale vs D12 Decision
    summary: the rationale says every hub converts without an operator step while the decision says such a hub needs a board.yaml edit before status writes work
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D10 (fold)
    summary: a fold leaves open what happens to the issue's GitHub labels and whether any are reported
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D6
    summary: D6 does not say what a path-only Board does with --label, nor which entries count when checking that no entry carries the label
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Notes for the plan writer, D5
    summary: the notes list importers of boardengine and a tally of two findings entries, which the writer stencil says not to gather and Plan-Write computes fresh
---

### [MEDIUM] import with an existing slug or an unknown into slug is unbounded

**Location:** `decision-record.md` D10, the `import` bullet.
**Issue:** D10 says `import` "creates a note" but does not say what happens when `slug` already names an entry.
D9 and D3 describe `upsert` as replacing fields on an existing entry, so a create routed through the same write could silently overwrite a task.
`into` naming no entry is likewise unstated.
**Fix:** State that `import` refuses a `slug` an entry already uses and an `into` that names no entry, each naming the way forward (pick another slug, or use `into`), before any board write or GitHub mutation.
Add an acceptance criterion.

### [MEDIUM] the retry of a recorded open issue is unbounded in three cases

**Location:** `decision-record.md` D10 Bound.
**Issue:**
D9 makes `issues` settable by hand through `upsert`, so a hand-recorded open issue is indistinguishable from an imported one whose close failed;
an explicit `import` of it comments and closes it as completed.
The Bound does not state that this is intended or what limits it.
It also does not say which slug is named when more than one entry records the issue (merging notes into a task passes through that state), nor what happens to an `into`, `title`, `brief` or `labels` that rides along with a retry, since only a payload slug is said to be ignored.
**Fix:** State that the retry closes any open issue some entry records, hand-recorded included, because recording it on the board is the claim that the board owns it, and that only an explicit `import` of that number reaches it.
Name the first recording entry in board order when several record it.
State that the retry ignores every other payload key.
Add acceptance criteria.

### [MEDIUM] D12 rationale contradicts D12's own decision

**Location:** `decision-record.md` D12, Rationale and the final Decision paragraph.
**Issue:** The rationale reads "every hub lyx runs in converts without an operator step", yet the decision says a hub with area labels needs one `board.yaml` edit before its entries can be written again, status writes included.
Conversion needs no step; writing does.
A plan writer reading only the rationale could drop the refusal message's naming of the file.
**Fix:** Rewrite the rationale so it claims only that conversion needs no operator step and nothing is lost, and say that the later config edit is the cost, bounded by a refusal that names the file and label and by D13 for loomyard.

### [LOW] fold leaves GitHub labels unstated

**Location:** `decision-record.md` D10.
**Issue:** Only the new-note path says how GitHub labels are carried and the dropped ones reported; the fold path is silent.
**Fix:** State that a fold carries no GitHub labels and reports none.

### [LOW] D6 leaves a path-only Board and the label scan scope open

**Location:** `decision-record.md` D6.
**Issue:** "A path-only Board does not validate the flag" does not say what it does instead, and "no entry carries it" does not say which entries count.
**Fix:** State that a path-only Board filters entries without validating, and that the check scans every entry of any kind and status.

### [LOW] Notes for the plan writer list importers and a tally

**Location:** `decision-record.md` Notes for the plan writer (last two bullets), D5.
**Issue:** The stencil bars gathering dependency lists; `Plan-Write` finds importers with its own lookups.
The battencli requirement is already a constraint in D3's Bound, so nothing is lost by removing the bullet.
D5's "two `cliTreeFindings` entries" pins a count that the acceptance criterion already states without one.
**Fix:** Delete the importer bullets and drop the count.
