# Report Claude usage outside the counted runs, W45 to W47

Adds a research report that prices every Claude Code session on this machine from 2026-11-03 to 2026-11-18 the way `tools/tokencount` prices a run.
It splits the usage by ISO week, session kind and model, and sets each kind beside the run spend tokencount reports.
Codex usage is out of scope, as the board entry says, and no code changes.

## What the report finds

- Tokencount counts $3,853.71 of the $4,975.37 priced; the other $1,121.66 is motif and raddle runs, the hub orch, unfinished runs, prime agents, other hubs, beam orchs and sessions outside LoomYard.
- The hub orch is counted in full when tokencount runs from the prime and dropped silently from a task worktree, where `CountOrch` reads the current directory's project.
- The rise from W46 to W47 is $237.5 to $474.7 per active day; counted runs give $168.6 of it, mostly tapestry runs, and usage outside them $68.6.
- Each kind that grew markedly has its reason in the report, and a closing table names the small change that would let tokencount count each kind.

## Design decisions

- The sweep is a one-off reader of `~/.claude/projects` that reuses `claudeengine`'s price table rows by value, so the figures match tokencount's pricing without touching its code.
  Its sources are not committed, because the board entry asks for no code; the report's method section states the rules instead.
- Weekly run spend is the counted runs' sessions priced by message date, because tokencount has no weekly view.
  On the shared population the sweep is within 0.8% of tokencount, and the report gives both figures.
- `claude-fable-5-2` has no price row, so its 108 messages sit in no dollar figure; the report says so and does not guess a price.

## Verification

- No test applies: the change adds a report and an index line.
- The sweep's totals were checked against `go run ./tools/tokencount -since 2026-11-03` from the prime: counted runs $3,853.71 against $3,823.94 + unpriced, hub orch $349.62 against $344.59 + unpriced.
