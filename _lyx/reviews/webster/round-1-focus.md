---
round: 1
focus:
  - "PATTERN.md `PATTERN-shuttle-stop` line: does the committed wording match card 11's instruction, which says to replace only the clause after the semicolon with an exact sentence, or does the diff rewrite the lead-in and the clause differently, and if so does the changed text still state the invariant correctly?"
  - "PATTERN.md `PATTERN-shuttle-stop` line: does the new text name exactly the loom driver's strand and the orch strand as exceptions, with the stated reason (no attach probe or Wait reads their records), and no other exception?"
  - "Does the diff touch anything beyond the one `PATTERN-shuttle-stop` line, such as Go code, other PATTERN entries or the `pattern/` background files, which card 11 says must stay unchanged?"
  - "Does the `PATTERN-shuttle-stop` entry still parse under the `internal/pattern` scans, and do the plan's `## verify:` commands over `internal/pattern` cover the change?"
---

Round 1 reviews a single-line documentation change.
The first question settles whether the reworded line departs from the card's exact text.
