---
round: 1
focus:
  - "internal/boardengine/runlock.go and raddlefile.go: does the run lock admit only the derived arras project shape (kind, cleared fields, unchanged fields, body prefix plus part list plus former body, `order` in `after` sequence with in-family dependencies first), only while the producer half of the status is Raddle-File, with filed parts read from `after` and never from the entry's `order`?"
  - "internal/boardengine/raddlefile.go: on a failed filing write, is the entry's prior record file restored and the filed parts' record files removed without removing the entry's own? Does the re-point of other entries' `depends_on` land in the same write, move only the arras slug onto the parts no other part depends on, and surface a board refusal as a halt naming the refusing entry rather than a return to Raddle-Write?"
  - "internal/loomshed/raddlefile.go: do `Call`'s store-done and AlreadyFiled repair paths write the mark, clear any run status, and skip the filing steps, with `finish` skipping EndEntry on every family path? Does an entry already of kind project that fails both tests halt naming `lym batten cancel` without sending the proposal back? Does Raddle-File write its own run status before the filing write when the status names another producer?"
  - "internal/shedrun/filingmark.go, internal/loomcli/wiring.go, raddlewiring.go and internal/battencli/commitstatus.go, cleanup.go: do loom's and batten's status writes skip the entry while the mark exists, and does the arras extension of `ClearMotifRun` remove the mark with a done arras run so the beam run's writes reach the entry again?"
  - "Tests against the acceptance criteria: is each D6 and D10 criterion reached by a test (failed board write, store write failing after the board write then resume, store-done without mark, foreign project halt, lock refusals, refused re-point), and are any added tests redundant with existing ones?"
---

Seed for round 1, set from the decision record's Goal, Decisions and Acceptance criteria and the diff stat.
