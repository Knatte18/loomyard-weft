# Fixer report — discussion round 2

## Changes

- **F1 (fixed as proposed).**
  Decision 8 now names the edge, an in-flight run that reuses a slug taking an older run's archive tag, and bounds it: a named slug whose worktree `<hub>/<slug>` still exists is in flight, so no tag is read and it takes Decision 3's no-status fallback.
  Premise checked: the package doc in `tools/tokencount/main.go` defines a run as finished once its pair is torn down, which removes the worktree.
  An acceptance criterion covers it, and the Open risks bullet is dropped since the bound closes it.
- **F2 (fixed as proposed).**
  Decision 1 now ends a turn at a user line whose text starts with `<local-command-stdout>`, and an acceptance criterion covers a local command typed between turns.
  Premise checked: most local commands write `"type":"system","subtype":"local_command"` lines, which Decision 1 already ignores, but `/model` writes `"type":"user"` lines (`-home-knatte-Code-loomyard-LYXHUB-gate-test-scope/aa43a0aa-….jsonl` lines 564–565), and 18 transcripts across the hub's project directories, the prime's included, carry such a user stdout line.
- **F3 (fixed as proposed).**
  Decision 8 states that both directories are compared as absolute, cleaned paths; the refusal's acceptance criterion says a trailing slash or relative spelling of the same hub is not refused.
- **F4 (fixed as proposed).**
  The Open risks copy of the mid-turn crash sentence is deleted; the Auto-mode assumption stays.

The support log gains a round 2 entry under Review rounds and Question ledger rows for the two new auto-picks.

## Disputed

None.

## Deferred

None.
