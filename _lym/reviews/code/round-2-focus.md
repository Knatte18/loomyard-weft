---
round: 2
focus:
  - "internal/creelcli/goclient.go `AwaitsReplyProbe`: the daemon refuses a program claim from a non-orch strand pane (`resolveSender`, internal/creelengine/identity.go). Settle whether the probe answers for a run waited on from such a pane, as `orchcli.notifyPrime` does with its strand claim first and program fallback on `ErrStrandClaimOutsidePane`."
  - "internal/shuttleengine/replyhold.go `releaseReplyHold` and internal/shuttleengine/wait.go hold clear: settle whether a run can finalize or evaluate its gate mid-turn on a reply that wakes the session without a turn start, given Claude's `ParseSessionSignals` reads a turn start only from `UserPromptSubmit` and creel's idle waiter is an `asyncRewake` Stop hook. Check that a test on an engine that parses session signals pins the answer."
  - "internal/creelengine/verbs.go `AwaitsReply` and internal/creelengine/sweep.go `releaseGoneHolder`: settle whether a later session reusing a removed strand's name counts the earlier session's open questions."
  - "internal/gatecli/cardfixer.go and internal/weavercli/wiring.go: settle whether each runner that waits on an agent run whose prompt renders the non-interactive parent directive carries a reply probe, since the directive says the run waits for the answer."
  - "Tests added for the hold in internal/shuttleengine/replyhold_test.go, internal/creelengine/awaitsreply_test.go and the four wiring tests: settle whether each pins a behavior the covering tests do not, including the hold clear after a release whose gate keeps the run going, the returned-unread and foreign-mail cases, and the pause paths."
  - "The Merge danger section of _lym/landing/summary.md: settle whether its blast radius names everything the diff can break, including the creel protocol verb and the parent-directive stencils."
---

Entries name where to look; the reviewer judges the current code afresh.
