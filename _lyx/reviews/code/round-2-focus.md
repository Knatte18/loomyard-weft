---
round: 2
focus:
  - "In internal/loomcli/decision.go, `decisionEntries` decodes the --from batch file: does a file with a misspelt or unknown key in an entry get refused with a message naming the key and a way forward, with nothing appended, and does a test reach that refusal?"
  - "In internal/loomcli/decision.go, `decisionEntries` after the array: is any data following the array, including a stray closing bracket or brace, refused with a way forward?"
  - "In internal/loomcli/decision.go, `beginAmendment` appends several titles for a --from batch and restores the amendment record when a later title fails: does a test make a later title fail and assert the record is restored, nothing appended and nothing committed?"
---

Targets the code paths the round-1 findings touched; the settled remainder needs no re-review.
