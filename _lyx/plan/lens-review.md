# Lens review

- test-economy 1 (LOW, `packFirstTurn` in the unavailable-index case): rejected — acceptance criterion 6 names a batch where quarry was unavailable, and the needle fails if the FRESH-READ fill ever comes to depend on pack content.
- test-economy 2 (LOW, first-turn needle in the no-member-glyph and none-resolved rows): rejected — same reason, for the criterion's other two cases.
- test-economy 3 (LOW, render-row needles repeating `wordingClaims` sentences): fixed — dropped "A span listed under several cards is read once." and the Edit-only needle from the multipart row.
- criteria-coverage 1 (LOW, `doc.go` criterion unpinned): fixed — the Intent states that review checks it.
- criteria-coverage 2 (LOW, locate row's `wantNone` lacks the earlier-card clause): fixed — added "an earlier card of this batch edited".
- forced-edits: no defects reported.
- row-reachability: no defects reported.
