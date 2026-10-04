---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-board-model-docs.md:36
    summary: D13 area-label list reaches the PR description only if the Describe session chooses to carry a doc.go paragraph from the diff; nothing in the card makes it a carry-worthy item
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/04-board-load-migration.md:34
    summary: The rule that an entry with neither tier nor kind loads as a note cites no decision, unlike the other two rules the card adds beyond D12
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/10-board-model-docs.md:40
    summary: ImpactSummary says a doc move with no code change but the card also adds the D13 label paragraph to doc.go
---

### [MEDIUM] D13 label list depends on the Describe session noticing a doc.go paragraph

**Location:** `_lyx/plan/10-board-model-docs.md`, D13 paragraph.
**Issue:** The card correctly sees that a commit message body never reaches the Describe session, whose sources are the decision record, `lyx board get <slug>`, the diff against the merge base and the run records (`landing-template-describe.md`).
The diff is in that set, so the channel exists.
But the session is told to cover what changes, `Closes #N`, design decisions and "anything to check or run by hand that was not run", and it is not told to copy package-doc text.
A `doc.go` paragraph written as a description of the migration reads as documentation, so the describer has no reason to lift the list into the PR body.
The decision record's D13 and its acceptance criterion require the PR description to carry the exact list.
**Fix:** Word the `loomyard's own area labels` paragraph as a pre-deploy hand step for the hub ("before deploying the landed binary, write this list into the parent's `_lyx/config/board.yaml`"), so it falls under the describer's "anything to run by hand that was not run" item.
State in the card that this is why the paragraph is phrased as an action.

### [LOW] Card 4's "neither tier nor kind" rule cites no license

**Location:** `_lyx/plan/04-board-load-migration.md`, the bullet "An entry with neither `tier` nor `kind` loads as a note with empty labels".
**Issue:** The card states the license for its two other additions, the absent-or-unknown type mapping (D3) and the dependency removal (D2).
This third rule gives only a purpose ("so a hand-written fixture still loads") and no decision.
D1 licenses it only partly: it says a new entry without `kind` is a note.
**Fix:** Cite D1 in that bullet, and say that the entry then has no type label and is refused on its next write by D3.

### [NIT] Card 10 ImpactSummary omits the label paragraph

**Location:** `_lyx/plan/10-board-model-docs.md`, `**ImpactSummary:**`.
**Issue:** The summary says the card moves the package doc with no code change.
The card also adds the D13 label paragraph, which the hub must act on and later remove.
**Fix:** Say the card is documentation only and that one `doc.go` paragraph is a temporary hub hand-off.

## Focus departures

None.
The three focus questions are settled as follows:

1. The derived list does reach the Describe session through the diff (F1 covers how to make that reliable).
2. Every rule card 4 adds beyond the decision record's rules is stated in the card's `**Intent:**`, and all but one name their license (F2).
3. The `import` bullet of card 9 states its bound: it acts only on an open, unrecorded, non-PR inbox issue, writes the board before GitHub, never edits or deletes issue content, never touches another repo, and a recorded issue is a no-op. This is not a defect.
