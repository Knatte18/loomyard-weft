# Card 5 — Document the time columns, idle and the Producers table

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
The scope's last item and its last acceptance criterion: the package doc, the report title and the launchers' description comments describe the time columns, the idle row, the Producers table and how each is measured.
Card 3 already changed the title the report prints to "Token usage and time by role".

The package doc in `main.go`, one sentence per line (`PATTERN-comment-line-breaks`):
- the opening sentence says the report gives token use and working time per run and per role;
- the weft paragraph says the default applies whenever the weft flag (`repositoryFlag`) is empty, that a `-hub` other than the current worktree's parent needs the weft flag named, and that each counted run reads its newest `archive/<slug>/<sha>` tag's shed `status.json` and `seed.json`, except a slug whose worktree `<hub>/<slug>` still exists, which is in flight; the calibration paragraph's statement of when the default applies is replaced by that one;
- a new paragraph, after the share column's, defines turn time (the turn-ending lines of Decision 1), a role's time as the union of its transcripts' turns clipped to the span with `+sub` its own row, the span (seed `started_at`, else the earliest transcript line, bounded by the next-older tag's commit time and the first entry's end, to the latest entry end, a folded entry ending at its `last_at`), the transcript-window fallback for a run without status, idle, the `time` and `time share` columns with the orch row's cells blank, and the "All runs" sums;
- a new paragraph defines the Producers table: each history entry's interval from the latest earlier end, its elapsed, idle and share, elapsed summing to the span and including any wait before the row started.
The launchers' header comments say the report holds token use and time by role, and that the time columns and the Producers table are described in the package doc of `main.go`.

**ImpactSummary:** comments only, in `tools/tokencount`; no code or test reads them, and the output path the launchers write stays.

**Commit:** 5: tokencount: document time by role, idle and the Producers table
