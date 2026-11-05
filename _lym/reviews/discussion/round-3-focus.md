---
round: 3
focus:
  - "Look at decision D3's double-fork start of the Slack relay against `PATTERN-reap-children` as the declared changes list words it (a child's starter waits on it by `cmd.Wait` or `proc.Reap`, bar a double fork's intermediate, which exits at once). Settle whether the record's `## PATTERN changes` section carries that change and D3 names the exception, or the double fork is replaced by a start that keeps the rule."
  - "Look at decision D5's handling of a creel read that the relay posts to Slack, against creel's 30-second read acknowledgement deadline (`ackReadTimeout` in `internal/creelengine/server.go`). Settle whether the relay stops posting before the deadline, whether its posted-id record is cleared only after a confirmed acknowledgement, and whether a backlog that outlasts the window posts each message once."
  - "Look at the decisions that introduce `lym slack login`, `lym slack setup` and `lym slack start` (D2, D3, D9). Settle whether each states who may run it, what it lets through if a strand runs it, and what bounds that, as the rubric's attack-surface question requires."
  - "Look at decision D5's sender filter, whose bound is that only orchs and lym programs reach the Slack channel, against `agentname.IsProgram`, which reports an EXTERNAL sender as a program. Settle whether mail stamped `<shortname>:EXTERNAL` is excluded."
  - "Look at decision D4's outcomes for a forwarded Slack message whose creel post fails or is refused (creel refuses a program claim from outside every pane while a live strand's pane pid is unknown). Settle whether the cursor stays, what is logged, and whether retrying is bounded."
---

Each entry names a site and a question; none caps a severity or pre-states a verdict.
Items outside these sites are reviewed under the ordinary rubric.
