---
round: 1
focus:
  - "internal/discussionparser/append.go: does AppendDecisions apply every entry in memory, write once, run Validate once, and restore the prior bytes on a finding or an error, with AppendDecision remaining a faithful one-entry case?"
  - "internal/loomcli/decision.go: are all batch refusals (--from with per-entry flags, unreadable or malformed file, unknown key, empty array, empty field, repeated title, repeated PATTERN entry ID) raised before anything is written, and does each name its way forward and have a test reaching it?"
  - "internal/loomcli/decision.go: with --amends-plan and a batch, is every title appended to the plan amendment record in the one commit, and do the restore rules hold on a failure after the record write?"
  - "internal/loomcli/decision.go: does a batch send each live receiver exactly one notice, short for ordinary receivers (opening phrase, every heading, the record, no inlined texts) and with every entry's text for the Merriam strand, while a one-entry call's notice and behaviour stay unchanged?"
  - "internal/loomcli/decision.go: does the success envelope keep heading and pattern_change for a one-entry call and add headings and pattern_changes for every call, and does the darn-run check confirm the committed record holds every entry of the batch?"
  - "Docs and stencil: do the verb's help, file doc, docs/overview.md and the orch stencil line name the batch form, matching what the code does?"
  - "Tests (append_test.go, decision_test.go): is each acceptance criterion covered without duplicating an existing test, and is any added behaviour left uncovered?"
---

Round 1 seed.
The focus follows the decision record's acceptance criteria against the six changed files.
