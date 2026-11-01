---
round: 2
focus:
  - "internal/shuttleengine/claudeengine/background.go `OutstandingForks`: does a fork tool_use line with no tool_result line yet count as outstanding, and is that case covered by a `TestOutstandingForks` row?"
  - "internal/shuttlecli/awaitforks.go and cli.go: are the await-forks envelopes (no strand, unresolvable strand, ready true, ready false with outstanding), `cobra.NoArgs` rejection and the verb registration each covered by a test?"
  - "internal/shuttleengine/await.go `Runner.AwaitForks`: are its early returns (engine without `OutstandingForkReader`, unknown guid) and the hand-off of the run's SessionID and paneCwd to the reader covered by a test, and does the FindRun error wording avoid asserting a cause?"
  - "The stencils that call the await-forks verb (burler-cluster-fan, burler-fix-self-check, twill-directive): does each give a bounded re-run rule and a fallback when the wait never turns ready?"
---

The focus entries target the places the first round found gaps in.
Each is a question to settle against the current diff, not a verdict.
