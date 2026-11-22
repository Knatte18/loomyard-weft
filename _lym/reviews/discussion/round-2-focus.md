---
round: 2
exclude_lenses: []
focus:
  - "D7 step 1 in decision-record.md: check the trigger for running `lym beam merge-parent` against `judgeOpenable` in internal/beamcli/pr.go, and settle whether any loop of drain, merge-parent and drain can run unbounded on a moving parent."
  - "D2 in decision-record.md: check which value a halt is judged under when `orchcli.ParentStrand` and `orchroute.FromPair` route it to an ancestor or the prime orch because the run's own orch is down, and settle whether the opt-out and the forced value both hold."
  - "D4 in decision-record.md: check the PR-Review path end to end against internal/landingshed/prreview.go (`askParentReview`, `gateRetryHint`), internal/landingshed/prsession.go (`sessionSpec`), internal/loomcli/prcontinue.go and the `parent-review` verb group in internal/loomcli/parentreview.go, and settle how each ask is sent, where its head is recorded, and what a re-gate or a mid-review toggle does."
  - "D5 in decision-record.md: check whether every halt outside or inside the five classes has a route to the operator or a settling action, and whether the class 5 repeat count reads a record that every settling verb writes."
  - "D3, D6, D8 and D9 in decision-record.md: check each introduced bypass, marker and mail path for a stated bound, and check that each store, reader and writer it names exists as described in the code."
---

Round 1 found two BLOCKING design findings and many MEDIUM design findings.
This focus points round 2 at the decisions those touched, and every lens still runs.
