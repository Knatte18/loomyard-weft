---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:47
    summary: Line numbers that locating an unresolved glyph returns are taken before the first edit, and no bound stops the fork from reading by them after an edit
    fix: In Decision 5, widen the replacement sentence from the listed lines to every line number obtained before the first edit, the pack's listed lines and the lines a lyx quarry resolve call or grep returned while locating an unresolved glyph, so after an edit the fork locates code by its text and never reads by any such line. Widen the matching acceptance criterion (the pack-on locate text says listed lines hold before any edit only) the same way.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:69
    summary: Open risks bullet 1 lists the first phase's width without the card files, which Decision 1 reads in the same phase
    fix: In Open risks bullet 1, change "context holds every resolved entry's span and every whole file Decision 2 reads" to "context holds every card file, every listed span and every whole file Decision 2 reads", keeping the rest of the bullet.
---

The round's gating question, the stale-read bound on Decision 2's batch-wide reads, is settled.
Decision 4's Bound names both sources of change, an earlier card's edit and a self-fix.
It names both kinds of pre-edit read: a listed span, and a whole file Decision 2 reads.
It says what catches each failure mode: an Edit's text match fails loudly, Decision 2's Edit-only rule covers a whole-file Write built from a pre-edit read, and the card and batch gates catch stale reasoning.
The implementer body does permit the Write tool (`webster-body-implementer.md`, "Paths you pass to the Read, Write and Edit tools stay absolute"), so the Edit-only rule closes a real gap.
The rule is scoped to files the fork itself edited in this batch, which is exactly the set that can go stale within the batch, since only the fork's own edits change the tree between its reads and its edits.
The premise "batches run one fork at a time" holds (`internal/websterengine/doc.go:5`).
Decision 2 points at Decision 4's bound, and an acceptance criterion pins the Edit-only rule.

### [LOW] Located line numbers escape Decision 5's guard

**Location:** `_lyx/discussion/decision-record.md:47`

**Issue:** Decision 2 moves locating every unresolved member glyph, with `lyx quarry resolve` or a grep, ahead of the first edit, batch-wide.
Both tools return line numbers along with the file.
Today the locate happens per card, right before that card's work, so its line numbers reflect every earlier card's edits.
Under this design, a glyph of card 3 is located before card 1's edit, and its line numbers go stale once card 1 or 2 edits that file.
Decision 4's Bound covers the whole file read after locating, but not the located line numbers.
Decision 5's guard sentence speaks only of "the listed lines", meaning the pack's lines.
So the one rule against reading by a stale line leaves out the line numbers this decision now takes early.
Reading by a stale number shows the wrong region, and Edit's text match still fails loudly, so the harm is a wasted or confusing read, not a silent bad edit.
That keeps this LOW, but it is a moved read-before-edit edge without a full bound, which focus item 2 asks about.

### [NIT] Open risks bullet 1 omits the card files

**Location:** `_lyx/discussion/decision-record.md:69`

**Issue:** The bullet describes how wide the phase before the first edit gets.
It lists the spans, the uncovered target and `Uses:` files, and the located glyphs' files.
It leaves out the card files, which Decision 1 also reads in that phase.
It covers every read Decision 2 adds, which is what focus item 2 asks.
The missing card files are a small consistency gap with Decision 1, not a coverage hole.
