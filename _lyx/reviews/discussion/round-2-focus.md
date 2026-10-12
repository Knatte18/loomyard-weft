---
round: 2
focus:
  - "In decision-record.md, the tackler decision (D7): settle what a wrong or stale tackler resume or goto can make Go do, and what enforces the bound beyond stencil text, comparing with the arbiter's consume path in internal/loomcli/arbiterconsume.go, which relaunches via shedengine.Relaunch with an Admit that checks the stop key and pause conditions under the status lock."
  - "In decision-record.md, the verify-failure goto (D4) and the stop table (D3): settle whether the failure record is matched as strictly as internal/battenshed/publishretry.go's recordMatchesHalt matches it, where the one-retry-per-run count lives given batten's publish-retry marker, and which table row a blocked stop with a non-matching record lands on."
  - "In decision-record.md, the hand-over decision (D5): settle whether the clean-tree guard is defined in the decision itself (neither shed-template-driver.md nor shed-template-driver-guide.md defines it), and what state a crash or failed spawn between releasing the driver lock and spawning the fresh driver leaves for a watched and an unwatched run."
  - "In decision-record.md, the driver seeding decision (D1): settle what happens when --child-driver is typed against an adopted seed that records no child_driver, given the stated precedence and PATTERN-driver-choice-single-site."
  - "In decision-record.md, the empty-reason paused stop decision (D13): settle whether internal/shedengine records a reason when it persists paused in Goto, on a bare pause_requested, or on a cancelled context, and whether the decision's premise holds against that code."
---

Rationale: each entry names a decision and the question to settle; none caps severity or pre-states a verdict.
