# Round 2 fixer-report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed.
  D4's Attach bullet now joins the spec's own `Requires` with `forks` when the run record carries `ForkSubagents`, the field `wait.go` audits on (`run.state.ForkSubagents`).
  D7's first bullet adds that `Wait` over a `ForkSubagents` record whose engine lacks the fork-audit capability fails the run naming the engine.
  The attach acceptance bullet covers a `ForkSubagents` record attached by a spec without it, and a new bullet covers the `Wait` failure.
- F2 (MEDIUM), fixed as proposed.
  D8 gives an attach refusal its own way forward: `lyx reed remove <strand guid>`, then re-step the row, the verb and wording `attach.go` already uses for a failed superseded-strand removal; no deny in `.claude/settings.json` covers it.
  D4's "respawn path is the operator's way forward" is replaced by a pointer to D8.
  D8 states that a finished run is harvested without the `Requires` check, its fork audit still following D7.
- F3 (LOW), fixed as proposed.
  R4 adds the changed-`engine:`-default divergence and names `codex-engine` as the task that resolves Webster's audit and parse engine per batch; D7's bound is reworded to hold only while the default is the one registered engine and no Webster spawn spec names an engine.
- F4 (LOW), fixed as proposed, picking the first option.
  D4's guid-less-methods bullet: an unresolvable `InterruptSettle` returns an error that tender's settle and the orch watcher log at Warn, then wait the longest settle they resolved, none when every resolution fails; an acceptance bullet covers it.
- `support-log.md`: a Round 2 entry, and ledger entries Q12 and Q13 for the two new picks.

## Disputed

None.

## Deferred

None.
