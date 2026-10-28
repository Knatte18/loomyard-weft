---
round: 1
focus:
  - "internal/fabricengine/refscanner.go: does the `lym gate` segment blanking stop at every shell separator (&&, ||, ;, |, newline) and apply only in command position, so a bare weft name in a later segment, a separator-prefixed weft path inside the gate segment and the worktree's exact sibling path still match (D1 bound, acceptance criteria)?"
  - "internal/websterengine/recoverbatch.go and auditledger.go: does recover-batch evaluate accept-audit's own evidence check through one shared function without mutating anything, and does each branch (HEAD past start, HEAD equal to start on a clean tree, no recorded start) name exactly the way forward D2's acceptance criteria list, never an accept-audit route on its own when the evidence refuses?"
  - "internal/websterengine/auditledger.go: does the refactor leave accept-audit's own refusals and the in-line read-only acceptance in recover-batch unchanged, with accept-audit's refusals still naming `accept-audit --batch NN` for the re-run and `ErrRecoveryNeedsFresh` still returned?"
  - "internal/websterengine/reset.go: does the run-lock refusal for targets other than `report-head` name the parent re-running the reset after the run stops as stuck, while `report-head` and the other guards are unchanged (D3)?"
  - "Tests: are the new cases rows in the existing table tests (PATTERN-test-speed), does each changed refusal have a test reaching it (PATTERN-refusal-way-forward), and do the scanner rows cover the chained-after-`git commit` form and the still-matching forms?"
  - "Docs: do internal/websterengine/doc.go and internal/fabricengine/doc.go and the scanner's doc comments describe the changed behavior, and does the change description's claims hold against the diff?"
---

Seed focus from the board entry and decision record; scope is the three decisions D1 to D3 and their acceptance criteria.
