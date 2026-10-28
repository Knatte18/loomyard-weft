---
round: 2
exclude_lenses: []
focus:
  - "In decision-record.md D10, settle whether every state a marked, unanswered message can be in has a stated path to an outcome: unread at a hook strand, unread at a pane-mail strand, read with the holder live, read with the holder gone, paused or dormant holder, held by another sender, and across a daemon restart. The sweep in internal/creelengine/sweep.go takes only unread mail and queued injects as subjects today."
  - "In decision-record.md D10 and D11, settle whether the chase clock and the chase count survive a daemon restart and a crash between the chase's posted record and its chased record, given that internal/creelengine/deliver.go persists an inject's consumed TTL only through aged records."
  - "In decision-record.md D2, D6 and D10, settle for each new routing edge (reply to a chase, program-signed mail made followable, chase body naming a sender) who can send it, who receives it and what bounds it, including off-Linux callers with no strand name."
  - "In decision-record.md D3 and D5, settle whether the question a held agent posts and the hold notice reach the same orch for an agent in a beam worktree, and whether an orch that follows either one clears the owed mark."
---

Focus is limited to the gating areas of round 1.
No lens is excluded: every lens that ran carries a MEDIUM or worse finding.
