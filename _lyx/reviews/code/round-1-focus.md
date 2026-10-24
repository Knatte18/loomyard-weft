---
round: 1
focus:
  - "internal/shuttleengine/wait.go, the `switch` at the held-turn-end site: settle whether the `usageLimited` case ordered before the `HoldDeferral` cases leaves the hold, the wait mark, the deadline and the API-error continue untouched, and whether a deferred hold already queued by an earlier turn end can still notify after a limited one."
  - "internal/shuttleengine/claudeengine/activity.go `readTurnLimit`: settle whether the newest main-chain turn end is picked as the decision record says (isTurnEnd or API-error marker, sidechain excluded), and whether `previousTurnEndAt` is the right bound for fork recency, including the no-previous-turn-end and unparsable-timestamp cases."
  - "internal/shuttleengine/claudeengine/activity.go `forkStoppedOnUsageLimit` and `newestAssistantIsUsageLimit`: settle whether a fork written before the previous turn end never silences a later notice, whether the `readBackward` callback handles chunk boundaries and partial first lines correctly, and whether a read failure always reads as no limit."
  - "internal/shuttleengine/hold.go and fakes_test.go: settle whether `UsageLimitReader` is optional as the record asks (an engine without it reports no limit), whether `heldTurnEnd.raw` is set at every construction site, and whether NotifiedOffset is left alone."
  - "Tests in hold_test.go and claudeengine/activity_test.go: check each acceptance criterion in the decision record has a row, and check test economy against the existing hold-notice and activity tests."
  - "Docs: the doc.go of both packages and the diff against PATTERN.md's provider-seam invariant (the `rate_limit` literal and transcript shape stay in claudeengine); check each doc claim against the code."
---

Seed focus from the decision record's acceptance criteria and the diff's three code areas: the Wait switch, Claude's transcript reading, and the optional capability seam.
