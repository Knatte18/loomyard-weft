---
round: 2
focus:
  - "internal/burlerengine/parentreview.go, AnswerParentAsk: when the locked update writes a state change and the record write fails, does the caller get the write error and never a refusal that claims the ask is now timeout?"
  - "internal/loomcli/parentreview_test.go and internal/burlerengine/parentreview_test.go: do the added rows each pin behavior no existing row covers, and does each assert on the written report or record rather than only on the envelope?"
---

The focus covers the one production change a fix makes and the new test rows.
