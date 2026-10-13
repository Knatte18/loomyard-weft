---
round: 1
focus:
  - "Fidelity: check each Decision and Constraint in `_lyx/discussion/decision-record.md` against the seven cards, and check that card 6 (`lyx loom fmt` verb, edit directive) and card 7 (worktree leftover retry) are licensed by the record."
  - "Attack surface and reachability: card 1 (skill-load turn awaited before color, retry and prompt), card 2 (readiness read from events before the pane), card 6 (new verb) and card 7 (bounded late-writer retry) — does each `**Intent:**` state what it can skip or let through and what bounds it, and does every prescribed refusal or timeout branch have a named input and test that reaches it?"
  - "Card order and shared files: cards 1 and 2 both edit `internal/shuttleengine/doc.go`, and cards 6 and 7 both edit `cmd/lyx/testdata/test-budget.yaml`; does any card depend on something a later card introduces?"
  - "Edit glyphs: on cards 1 and 3, do the `Edit` members carry the re-sign arrow wherever the `Intent` changes parameters, results or receiver, and is any non-test file listed by path only while the `Intent` names symbols?"
  - "Test economy and granularity: card 1 touches several existing tests and fakes (`skillReed`, `hookStateSkillEngine`); does it plan new tests for behavior `TestStartup_SkillLoadTurns` or another existing test already covers, and are private helpers split off needlessly?"
  - "CLI shape: card 6's `lyx loom fmt` against the audience bullet of `pattern/PATTERN-cli-cobra.md`."
  - "Verify coverage: does `## verify:` run the tests of every package the cards target, including `tmuxkit` (card 5) and any `integration`-tagged tests, and is the section in the template's order?"
---

Focus is on the cards that add behavior or a verb (1, 2, 6, 7); cards 3, 4 and 5 are small and drew one entry at most.
