## Goal

```markdown
## Seen (`docs/research/turn-savings-measure-2026-11-04/report.md`)

The batching breach hook (`internal/shuttleengine/claudeengine/batchingnudge.go`) fired 4 times in 24 landed runs.

- Its lookup chain matches Read, Grep and Glob tool calls only. lym sessions search with Bash `grep`: the sample holds 1 Grep and 1 Glob call against 3,040 Bash calls, so the lookup-chain breach, 9.8% of context by the report's class reading, never fired.
- It skips every call from a subagent or fork, while forks hold 57 of the sample's 117 edit-chain turns and most lookup chains.

## Wanted (operator, 2026-11-04)

1. A Bash call whose command is a read-only lookup (`grep`, `rg`, `find`, `ls`, `cat`, `head`, `tail`, `sed -n`, `wc`, a `git` read such as `log`, `show`, `diff` or `status`, and a pipe or `;`-chain made only of these) counts as a lookup in the lookup-chain breach, as Read, Grep and Glob do. A Bash call with any other command breaks a chain as today.
2. The hook reaches subagents and forks: a call carrying `agent_id` is judged against that agent's own transcript, context and nudge count, under the same `batching_nudge` thresholds.
3. The report's method stays valid for a re-measure: `docs/research/turn-savings-measure-2026-11-04/run-all.sh` keeps working on runs that land after this change.

Thresholds are unchanged.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - bash-early-return-unpinned (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
