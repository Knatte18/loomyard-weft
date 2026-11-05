---
round: 1
focus:
  - "Fidelity to the answer key: read every D1-D9 bullet and Acceptance criterion in `_lym/discussion/decision-record.md` against the cards' Intents. Settle whether the 5-minute creel-refusal answer, the 24-hour skip, the 5-refusal `ok: false` cap, the ack-deadline record and the `cmd/lym/daemonwakeups_test.go` loop registration are each carried by a card."
  - "Card order across cards 3 to 14: settle whether any card's `Uses` or Intent depends on a symbol a later card creates (for example card 4's geometry and teller against card 2's names, cards 7 to 9 against card 6's client, and cards 11 to 13 against card 9's `EnsureRelay`)."
  - "Attack-surface bounds: for the pane-ancestry refusals (login, setup, start, away, back, stop), the `EnsureRelay` start that never refuses for a caller's pane, and the program-claim-in-strand-pane refusal in card 3, settle whether each Intent states what it lets through and what bounds it."
  - "Reachability of prescribed refusals: for each refusal or guard branch a card prescribes (unknown-pane refusal, missing token or setup file, first-read refusal ending the relay versus retrying, unknown beam slug, over-4-KiB body), settle whether the card names an input and a test that reach it and that no upstream check rejects."
  - "Test economy in cards 4 to 10: settle whether any card plans one test per symbol or a test an existing test already covers, or asserts static text such as help text, the orch-role prose or the PATTERN lines. The `Review:` criteria on D2 help text and D7 are met by code review, not tests."
  - "Edit-card typing in cards 2, 3, 5, 9 to 13: settle whether `Edit` targets list member glyphs where the Intent names symbols, whether the re-sign arrow is present where parameters or results change, whether `ImpactSummary` carries a real blast-radius conclusion, and whether any `Extends` ref is one the card rewrites."
  - "Granularity and PATTERN fit: settle whether any card is one per literal symbol or bundles unrelated surfaces, and check each card against the `PATTERN.md` entries it touches (`PATTERN-cli-cobra`, `PATTERN-cwd-resolution`, `PATTERN-child-processes`, `PATTERN-agent-addressing`). Also check card 1's PATTERN texts against the overview's two PATTERN changes."
  - "Research steps and board or task writes: settle whether any card leaves a design-relevant fact to be found while it runs, edits this task's own board entry, or files or promotes a task. Settle whether any `**Verify:**` line names a raw test command instead of a `lym gate test` call."
---

Seed focus only; no round has been judged.
Entries name where to look and the question to settle, and exclude the `Do not flag` list (gate-checked rows, derived dependency edges, `ImpactSummary` on `Rename`, `Move`, `Prosa` or `Custom`).
