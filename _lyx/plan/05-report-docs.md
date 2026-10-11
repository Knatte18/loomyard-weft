# Card 5 — report-docs

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
Describe the time columns, the idle row and the Producers table, and how each is measured, in the package doc and the launchers' description comments (the record's doc scope and Decision 10).
This card changes comments only; no code line changes.

`main.go`'s package doc, one sentence per line (`PATTERN-comment-line-breaks`):
- The opening sentence says the report gives each run's token use and time per run and per role.
- The paragraph on finished runs and -weft says the archive tags are read on every invocation, and that the -weft default, the current worktree's weft sibling, applies whenever the flag is empty, so outside a resolvable worktree the tool needs -weft.
  It says a `-hub` other than the current directory's parent with -weft empty is refused, both compared as absolute, cleaned paths.
  The calibration paragraph that says the default applies "whenever the flag is empty and either no slugs are named or the calibration is asked for" drops that condition.
- A new block after the weight, share and cost paragraphs states the time model:
  - a transcript's working time is the intervals between its consecutive timestamped user and assistant lines, except an interval after a line that ends the turn: an assistant line whose `stop_reason` is set and is not `tool_use`, or a user line starting with the interrupt marker `[Request interrupted by user` or with `<local-command-stdout>`;
  - a foreground tool call is working time, since the session stays in its turn;
  - a role's time is the union of its transcripts' working intervals clipped to the run's span, `<role>+sub` its sub-agents' union, and the `time share` column divides by the table's span;
  - the run's span runs from the seed's `started_at`, or the earliest transcript line without one, to the latest end among the archived shed history's entries, a folded entry ending at its `last_at`;
    the start is never earlier than the commit date of the slug's next-older archive tag and never later than the first entry's end;
  - the status and seed are read from the newest-committed `archive/<slug>/<sha>` tag through `gitrepo`'s read side;
    a run with no tag, or whose worktree still exists, uses its transcript window as its span, and its section says no shed status was read;
  - the `idle` row is the span minus the union of every working interval of the run's own transcripts and sub-agents, orch transcripts excluded, so parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push count as idle;
  - the orch row's time cells are blank, since orch spans many runs;
  - the Producers table splits the span by history entry, each entry owning the interval from the latest earlier finish to its own, so elapsed sums to the span and includes any wait before a row started, and its idle is the part no transcript works in;
  - "All runs" sums each run's role times, idle, spans and Producers rows, with no union across runs.

`tokencount.sh` and `tokencount.cmd`: their opening comment calls the launcher one for the token and time report, and the rest of each comment stays.

The descriptions name rules, not the report's current rows or counts.
