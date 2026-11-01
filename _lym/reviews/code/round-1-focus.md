---
round: 1
focus:
  - "internal/shellguard/rows.go, Rows and refuseRawRow: the answer key keeps the full gate rule in the raw go row's notice and in every steer; check that the raw go row's notice still carries the whole rule and that refuse-raw's steer is unchanged apart from the refactor."
  - "The refuse-raw notice no longer states the gate rule; settle whether it still gives a session enough to act on when the refuse list is non-empty, and whether it stays within the answer key's 'says only what it refuses'."
  - "Where Row.Notice is rendered into the appended system prompt (search the shellguard consumers): settle whether the refuse-raw notice is always emitted together with the raw go notice, so the rule is stated once and never zero times."
  - "internal/shellguard/rows_test.go TestCheckTexts: settle whether the exact-string notice assertion and the 'Gate rule' notice count duplicate coverage or assert static prose, and whether the count test would still catch a regression."
  - "internal/shellguard/doc.go: settle whether the new paragraph is accurate against rows.go and whether the repo's constraints document asks for further docs to land with this change."
---
