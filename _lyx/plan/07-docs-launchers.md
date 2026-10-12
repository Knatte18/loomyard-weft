# Card 7 — docs and launchers

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:** The package doc in `main.go` and the description comments of the two launchers describe what the report now holds, in one-sentence lines per `PATTERN-comment-line-breaks`.
The package doc's opening paragraph says the report covers token use and time per role; a new paragraph defines working time as turn time (the turn-ending lines of Decision 1, a foreground tool call included), a role's time as the union of its transcripts' working intervals clipped to the run's span with sub-agents under `<role>+sub`, and the `time` and `time share` columns after `share`, the orch row's time cells blank.
It defines the span (seed `started_at`, or the earliest transcript line bounded by the next-older archive tag and the first entry's end, to the latest history entry end, a folded entry ending at `last_at`), the `idle` row (time no session of the run is in a turn, orch sessions excluded) and the Producers table (elapsed as the gap between finish times, idle, share; summed in "All runs"), and that a run with no archived status uses its transcript window and prints no Producers table.
It states that `status.json` and `seed.json` are read from the newest `archive/<slug>/<sha>` tag of the weft, that a named slug whose worktree still exists reads no tag, and that the weft default now applies whenever the weft flag is empty, with a hub flag naming another hub refused until the weft flag is named.
The sentence that limits the weft default to no slugs or the calibration is replaced.
`tokencount.sh` and `tokencount.cmd` change the description lines only, from the token count to the token and time count; the output path, the commands and the arguments stay.

**ImpactSummary:** Comments only: the package doc and two launcher headers change, no code, no output path and no flag, and `docs/overview.md` lists no tokencount row to update.
