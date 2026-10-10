---
round: 1
focus:
  - "D10 in decision-record.md: the teardown claim clearing removes the `board-lock-running` hold on an entry. Does the Decision state what it can now let through (a scope change on an entry whose run was torn down by hand while still live) and what bounds it, beyond the not-done and run-form conditions?"
  - "D9 in decision-record.md: Finalize now commits the parent weft's `_lyx/shed/` subtree before the merge. Does the Decision state what that commit can sweep in (a sibling run's mid-transition records) and what bounds it, given the guard stays strict for every other path?"
  - "D3 and D4 in decision-record.md: the `PR-Gate` to `Darn` edge re-spawns a writer. Settle whether the bound on the whole path is stated end to end (five rejections times the verify budget per spawn), and whether the verify gate budget resets on a re-spawn."
  - "D6 and D7 in decision-record.md: the darn gate and Publish run an operator-configured shell command. Settle whether the trust class and the empty-command refusal fully bound it, and whether D6's behaviour while a Publish failure record is present is a Decision or only an assumption."
  - "D8 in decision-record.md: log pruning deletes evidence. Settle whether the 'unless a Publish failure record names them' exception is bounded, for example against a stale failure record pinning logs without limit."
  - "Completeness-before-leanness: for each relocation candidate, check first that the requirement it carries already sits in decision-record.md, since Plan-Write never reads support-log.md. In particular, confirm that nothing a Decision depends on lives only in support-log.md's Interview or Question ledger."
  - "Consistency between Decisions, Constraints, Auto-mode assumptions and Acceptance criteria: each acceptance criterion should trace to a Decision, and each Auto-mode assumption should name a choice a Decision actually makes."
---

Seed focus only: it names where to look and which question to settle, not verdicts.
The "Notes for the plan writer" subsection, absent rejected alternatives in the decision record and incomplete cross-reference enumeration are not flagged, per the rubric.
The file and symbol pointers in the plan-writer notes are not flagged as over-gathering; they are optional by contract.
