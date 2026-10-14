---
round: 2
focus:
  - "Round numbering in the checklist block of decision-record.md D2: settle whether a review-row history entry that opens no Burler round (BurlerReviewProducer.Call in internal/shedadapters/burlerreview.go returns a BudgetExempt Stuck without spawning when the highest complete round is unjudged or a restart finds the review accepted) is placed in the round already open, and whether the rule maps entries to round numbers through the package that owns the round file names rather than a second parse."
  - "Migration in decision-record.md D11: settle what happens to a string discussion_advisors list in documents that never set discussion_producer, namely local files, sparse worktree files and a task's run_settings, which configsync.reconcileSparseFile and configreg.PrepareRunSettings pass through MigrateConfig before validation. The question is whether any path silently empties or misreads the list instead of refusing it with the stated way forward."
  - "Reach of discussion_advisors in decision-record.md D7 and D8 once discussion_producer is retired: settle whether the Decisions state what a non-empty list can now start in a local override or run_settings, what bounds it, and whether the D8 claim about bounding no more than the replaced key holds for advisors as well as lens forks."
  - "Arbiter attempt reading in decision-record.md D3: settle whether the stated reader (loomengine.ListArbiterAttempts in internal/loomengine/arbiter.go fails the whole listing on one unreadable attempt.json, and Number and StopKey live in that file) can produce the unreadable-attempt line the Decision requires, and how an attempt with no verdict past its recorded Deadline is shown."
  - "Placement of Bouncer entries that route nowhere (an awaiting escalation or a halt) in decision-record.md D2 against D3: settle whether the two Decisions agree on the round such an entry and a following arbiter line appear in, given the run can continue after an arbiter continue."
---

Focus targets the findings that change a decision or an approach.
Local wording and bound statements are left to the ordinary review.
