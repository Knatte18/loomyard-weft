---
round: 1
focus:
  - "Card 4, `awaitStartup` backoff: settle whether the probe reaches the `dismissalGap` ceiling, whether gate-key spacing of at least one gap still holds when the poll interval shrinks, and whether the tests inject the run clock instead of waiting out an interval."
  - "Cards 3 and 4, `PATTERN-completion-signal`: settle whether every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`, and whether `completionsignal_enforcement_test.go`'s counts are unchanged."
  - "Card 1, `PATTERN-shuttle-provider-seam`: settle whether the lone `%` knowledge stays in `claudeengine` with no Claude specifics added to `shuttleengine`, and whether a box holding `%` plus other text is still read as non-empty."
  - "Card 2, `Result.StartError`: settle whether every path that builds a not-started `Result` sets it, and whether the single-LLM producer's failure names the cause without losing the existing error text."
  - "Card 3, never-ready teardown error: settle whether the pane tail comes from the last capture, is bounded, and is absent when a cause is already named."
  - "Card 5, `holdNoticeMaxDeferral`: settle whether the 10-minute value is reflected in every doc, comment and test that stated 20 minutes."
  - "Test economy: settle whether any new test duplicates an existing test's coverage, and whether any added behavior has no coverage."
---

Focus is set from the plan's cards and shared decisions only; no diff finding is pre-judged.
