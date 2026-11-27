---
round: 1
focus:
  - "Cards 2-11, closed import set and order: read each moved declaration, its field types, method bodies and referenced constants in the code. Does each compile inside internal/fabriclayout (standard library, lymdirs, weftname, no banned I/O import) at the card that moves it, or does one need an engine name that moves in a later card or never moves, in particular among the result types of cards 7, 8, 10 and 11?"
  - "Cards 2-11 against D1, D7 and the overview's 'what the layout takes' decision: compare the Move lists with every exported engine name that files outside fabric use. Is a name of D1's kinds left behind, is a moved one not of those kinds, and does a file that should drop its engine import keep it for such a name?"
  - "D7 and the second acceptance criterion: search for every scan, guard table, owner map, allowlist, //lym:seam or //lym:watch line, PATTERN background, doc.go and docs file that names a moved symbol or its file. Does the card that moves the symbol carry each one?"
  - "The added entry 'The restructure cuts fabric's test code', with D5 and D6: weigh the test code the plan adds against what it deletes. Do the foundation's own tests shrink, are there duplicate or redundant tests the cards keep, move or newly plan (read the existing tests before relying on coverage), and does each planned deletion leave its assertion held where D5 requires?"
  - "Cards 12, 14 and 15 against D2 and D3: trace one rule through Snapshot, Answer, runRule and snapshotReader.read. Which code supplies each snapshot field, including BackoffDelays and a ReadFailure's Field, Store and Class; can the engine hold today's locks and re-read where D3 says; and does the reader start no more git processes than today's reads?"
  - "Card 14 against card 15: are the signatures and contracts between them (snapshotRead, error wrapping, how a failed read is attributed to a field and store) specified well enough that two separate implementers produce code that agrees?"
  - "PATTERN fit for cards 12-15: read PATTERN.md and the backgrounds for test stand-ins, test-only code, test isolation, code conventions (including vocabulary and values declared in both fabriccore and fabriclayout) and refusal way forward. Is each card within them, and is the overview's stated departure for dead-code allowlist entries licensed and bounded?"
  - "Cards 1, 6, 12, 14 and 15, attack surface: for each new or widened guard, allowlist entry or test accessor, does the card's Intent state what it can let through and what bounds it? Cards 1, 2 and 12: do the two leaf tests fail for every import the first acceptance criterion names?"
  - "Cards 6 and 9, typing and granularity: their Edit targets are package glyphs while the Intent names specific symbols, signature changes and removed declarations. Check this against the card format and the rubric's member glyph, re-sign arrow and granularity bullets."
---

Seeded from the overview, the cards and the decision record.
No source code was read, so each entry is an open question for round 1 to settle against the code.
