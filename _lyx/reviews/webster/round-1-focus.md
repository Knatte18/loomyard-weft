---
round: 1
focus:
  - "Card 28 (webster-pause-retire): confirm every remaining reference to webster's pause verb, flag and paused outcome is gone from code, CLI help, docs and tests, and that no deleted path leaves a caller or a doc.go claim behind."
  - "Cards 5 and 6 (network holds): settle whether every remote subcommand in gitexec's runCore and every request through githubclient's authRT.RoundTrip reaches pauserecord.Network, and that the no-gate case stays a no-op."
  - "Cards 8, 16 and 19 (idle clocks and deadlines): settle whether each clock measures awake unpaused time through pauserecord.AwakeSince, with no leftover running-total or wall-clock path."
  - "Cards 7, 14, 15 and 27 (covered refusals and lock yield): settle whether each covered-slug path refuses or yields as the plan states, and that an unreadable record reads uncovered everywhere."
  - "Per-card mechanical checks: for each Rename or Delete group in the cards, settle whether its type-specific check ran against every target of the group."
  - "Test economy: settle whether any new test repeats the coverage of an existing test, and whether any behavior added by the diff has no test."
---

Seed focus for round 1, set from the plan's Card Index and Shared Decisions.
