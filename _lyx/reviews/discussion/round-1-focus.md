---
round: 1
focus:
  - "decision-record.md D1: the helper holds the handle lock across the caller's whole read and reruns it. Settle whether a read that calls another gitrepo read (HeadContains using SHAExists, merge-state reads) can re-enter the lock or retry twice, and whether the 'at most once per call' bound still holds when reads nest."
  - "decision-record.md D2: the bound covers gc.auto=0 for operators, but check what housekeeping can now skip or let through: gc --auto prunes unreachable objects past git's expiry while another pair's step runs. Settle whether the stated bound (git's default expiry, D1 retry) covers every read path, including reads by agents' own git CLI in other pairs, or only go-git reads."
  - "decision-record.md D3: the 10-minute deadline and low-speed limit now turn slow legitimate clone, fetch or push into a transport failure that existing paths treat as transient. Settle whether the Decision states what a transient mark then does (retry loop, or stop), so a bounded timeout cannot become an unbounded retry."
  - "decision-record.md Acceptance criteria against Decisions: check each Decision's behavior has a checkable criterion. Candidates: D2 picking up gc.auto=0 on an existing hub at its next removal (Q3), and D3 adding pull to IsTransportFailure."
  - "Both files, relocation: check whether decision-record.md holds deliberation or snapshot measurements that carry no requirement for the plan writer (the object-count figures in D2's rationale; D2's repack-under-another-pair point repeated in Open risks). Apply the completeness-before-leanness test before proposing any move."
  - "Cross-file consistency: Q6 says the #447 note's reuse expectation is recorded as an open tension in the decision record, while the record lists it only as an auto-mode assumption. Settle whether the record and ledger agree on where the tension lives and what the plan writer must do about it."
---

Focus lists questions to settle; none pre-states a verdict.
Items checked against the rubric's Do-not-flag list: none targets a missing "Notes for the plan writer", rejected alternatives in the record, or call-site enumeration, and none asks for signatures or file:line facts the writer stencil says not to gather.
