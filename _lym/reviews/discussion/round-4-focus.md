---
round: 4
focus:
  - "Finalize past its parent-side merge in `_lym/discussion/decision-record.md` (D1 act rule, D2, D4, D6 step 4, D9): settle whether the record makes the merge commit reach origin before origin's records say past-merge, or states how the adopter finishes the landing without it, and what becomes of the old owner's local parent branch after a hand-over. The code in `internal/landingshed/finalize.go` merges locally, marks the board entry done, then pushes the parent branch; `internal/fabricengine/replay.go` reads the local parent branch for past-merge."
  - "Pair push order in D2 and D4: settle whether the records-before-code order holds for every caller of `fabricengine.PushPairAnchored` (the transition push, `lym loom commit-records`, Finalize's task push, the fabric push verb, the beam push) or only for shed transitions, and whether D4's claim about origin's code branch holds for each caller."
  - "Cancelled batten records in D3 and D1: settle which step removes a cancelled run's record on every path that ends its pair (`lym batten cancel --remove` through `removePairAndBranch` in `internal/battencli/cancel.go`, the dobby teardown, `lym fabric remove`), and whether a released task whose pair is gone reads queued."
  - "Added-after-Discussion entries against the body: settle whether the main decisions, Constraints, Scope, Auto-mode assumptions, Open risks and Acceptance criteria agree with every Added entry, so no section still specifies a verb or mirror an Added entry removed, and no Open risk asks for a call an Added entry settled."
  - "D2 network admission and D7 upgrade hold: settle whether a wait on a held teardown lock inside the synchronous push is bounded so a pause reaches the process, and whether an upgrade hold is written only when a subject remains to re-run."
---

Focus targets only the changed D1 to D11 text and the folded Added entries.
