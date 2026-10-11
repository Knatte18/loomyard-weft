# Card 6 — time-docs

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:** Describe the time measurements in the package doc and the launchers (the doc acceptance criterion of the decision record).
This card changes comments only.

`tools/tokencount/main.go`'s package doc, one sentence per line (`PATTERN-comment-line-breaks`) in every sentence it adds or rewrites:
- the opening says the report gives token use and time per run and per role;
- the -weft sentences say its default, the current worktree's weft sibling, applies whenever the flag is empty; that a `-hub` other than the current worktree's parent with -weft empty is refused, the two compared as absolute, cleaned paths; and that with -weft empty the tool fails outside a resolvable worktree;
- a new paragraph after the weight and share paragraph defines the `time` and `time share` columns: turn time, from a session's prompt to the end of its turn, ended by an assistant line whose `stop_reason` is set and not `tool_use`, by a user line starting with the interrupt marker `[Request interrupted by user`, or by a user line starting with `<local-command-stdout>`; a role's time is the union of its transcripts' working intervals clipped to the run's span, `+sub` its own row; the orch row's time cells are blank; time shares divide by the run's span, or by the summed spans in "All runs", and can sum past 100% where sessions overlap;
- the same paragraph defines the `idle` row: the span minus the union of every transcript's working intervals in the run, orch sessions and orch sub-agents excluded, so parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work count as idle;
- a paragraph defines the span: read from the run's newest `archive/<slug>/<sha>` tag by committer date, it runs from the seed's `started_at`, or the earliest transcript line, bounded below by the next-older tag's commit date and above by the first history entry's finish, to the latest entry finish, a folded entry finishing at its `last_at`; a run with no tag, no status at it or no history, and a named slug whose worktree still exists, use the transcript window and say no shed status was read; an archived seed or status that fails to decode aborts the report naming the slug, the tag and the file;
- a paragraph defines the Producers table: each history entry owns the interval from the latest earlier finish to its own, so a row's elapsed is the gap between finish times and includes any wait before the row started; per producer it gives entries, elapsed, idle and share, the elapsed summing to the span, and "All runs" sums it over the archived runs.

`tokencount.sh` and `tokencount.cmd`: the opening description comment says the report holds token use and time by role, with the idle row and the Producers table, and keeps the fixed output path.

**Commit:** 6: tokencount: document the time columns, idle and Producers
