---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (Decisions, D4 — pack-on fragment sentence)
    summary: D4 tells the fork to read an unresolved member glyph's file whole in the turn right after the card read, but that file is known only after a locate call that itself needs that turn
    fix: In D4's pack-on fragment sentence, remove ", an unresolved member glyph once located" from the parenthetical so the turn-right-after list reads "(a plain path, a self glyph's file)", and add after that sentence one new sentence to D4 reading "An unresolved member glyph's file is read whole in the turn right after the `lyx quarry resolve` call or grep that locates it, still before any edit."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (Open risks; Decisions D3; Acceptance criteria, card-read bullet)
    summary: The one Open risk rests on a false premise, while the find-by-signature rule's real gap, a member an earlier card of the batch re-signed, goes unstated and unbounded
    fix: In D3, change "and the member is found in it by its listed signature." to "and the member is found in it by the member name its glyph names, its listed signature serving only as a cross-check, since an earlier card may have re-signed it." In D3's description of the locate fragment, change "the fork finds the member by its listed signature" to "the fork finds the member by its glyph's member name, with the listed signature as a cross-check". In Acceptance criteria, change "keeps the find-by-signature rule for a file edited earlier in the batch" to "finds a member in a file edited earlier in the batch by its glyph's member name, with the listed signature as a cross-check". Replace the Open risks bullet with "- None."
---

### [MEDIUM] D4's turn-right-after read includes a file not yet known in that turn

**Location:** `_lyx/discussion/decision-record.md`, D4.

**Issue:** The first focus question is whether the pack-on FRESH-READ text asks the fork to read a file before its name can be known.
For the card-named files it does not: D3 reads the card file and the listed spans in turn 1, and D4 defers every file only the card file names (a plain path, a self glyph's file) to the turn right after, by which time the card file has been read.
Every such file is read whole before the first edit, so that half of the question settles as not a defect.
The one exception is D4's third parenthetical case, "an unresolved member glyph once located".
A member glyph names a package, not a file (the locate fragment says so), and the pack lists an unresolved member with a reason and no file.
Its file is known only after the `lyx quarry resolve` call or grep that the locate fragment prescribes at step 2, which runs after step 1's turn.
So D4's "in the turn right after, before any edit, reads whole every file … (… an unresolved member glyph once located)" asks for a read in turn 2 of a file whose name turn 2's own locate call produces.
A plan writer who renders the sentence as decided ships a stencil instruction the fork cannot obey for that case.
"Before any edit" is satisfiable, and the acceptance criterion pins only that, so the fix is to drop the turn pin for this one case.

### [LOW] Open risks names a risk that cannot occur and misses the one that can

**Location:** `_lyx/discussion/decision-record.md`, Open risks, D3, and the card-read acceptance criterion.

**Issue:** The second focus question is whether D3 and D4 handle a span in a file an earlier card of the batch already edited, and whether Open risks states only a residual risk no Decision bounds.
D3 handles the shifted span: such a file is read whole in the card's first turn, not by offset, and D4 makes that whole-file read the card's fresh read, so the fresh read is never of shifted lines.
That half settles as not a defect.
The Open risk does not hold up, though.
D5 keeps a `var` or `const` signature's first line, which D5's own rationale says reads `var name …` or `const name …`, so it carries the member's name.
Two members of one file with the same first line therefore share a name, and a package admits one declaration per name, so the confusion the risk describes cannot arise.
The find-by-signature rule does have a real gap that no Decision states or bounds.
The plan spec's `Edit` re-sign arrow changes a member's signature, so when an earlier card of the batch re-signs a member a later card also names, the later card's pack lists the batch-start signature, which no longer appears in the file.
The glyph's member name still finds the member in the whole-file read D3 already requires, so making the name the key and the signature a cross-check closes the gap and leaves no residual risk to record.
