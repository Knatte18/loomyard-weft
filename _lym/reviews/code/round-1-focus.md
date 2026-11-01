---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/background.go `isBackgroundLaunch`: it now counts an Agent or Task launch with `subagent_type: fork` as background work. Settle which existing callers read this (the session-state and outstanding computation, the pause and idle decisions) and whether a fork launch now changes their behavior for a session that does not call await-forks, and whether the tests cover that."
  - "`OutstandingForks` and `unfinishedForks` against the transcript reading they reuse: settle whether a fork launch is matched to its completion by the task id the launch's tool result returned, whether a launch whose result is not yet written counts as outstanding, and whether the wait can return ready true early or never return ready true."
  - "`AwaitForks` and `awaitForks` in internal/shuttleengine/await.go: settle the cap, clock and context handling, the error paths (unknown guid, engine without the capability, transcript read error) and whether the test uses the told clock the way the file-poll tests do."
  - "internal/shuttlecli/awaitforks.go and the `reed` field in cli.go: settle that the verb resolves the calling strand correctly, that its ok and error envelopes match the sibling verbs, and that it is registered and reachable with the audience annotation its role expects."
  - "The three stencils (burler-cluster-fan, burler-fix-self-check, twill-directive): read each against the decision record's Goal, which asks for one wake for all forks while the forks stay parallel. Settle that the wait is placed after the spawn and after the lead's own holistic work, that no rule the stencil already carried (write nothing while a fork is outstanding, save lens reports verbatim) is dropped or weakened, and that the three wordings agree."
  - "The Goal says to measure with the `fork-wait` turns per session. Settle whether the diff leaves that measure unchanged or broken, and whether docs/overview.md and the package docs state the new verb accurately."
  - "Commit messages: the range's trailers and any Pattern-Departure, checked against the PATTERN entries the change touches."
---

Seeded from the committed diff and the decision record's Goal; the record's other sections are empty.
