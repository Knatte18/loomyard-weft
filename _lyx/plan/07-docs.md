# Card 7 — docs

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:** Describe the time measurement where the tool describes itself, the last scope item.
The package doc in `main.go` gains, after the paragraph on roles, the time model: working time is turn time between consecutive user or assistant lines unless the first ends the turn, the three turn-ending lines, a role's time as the union of its transcripts clipped to the run's span, the `+sub` rows, the `idle` row as the span minus every transcript's working union with the orch sessions never in it, the span from the seed's `started_at` or the earliest transcript line, bounded by the next-older archive tag and the first history entry's end, to the latest entry end, the Producers table with a row's elapsed being the gap between finish times, the "All runs" sums across runs, the archived `seed.json` and `status.json` read from the newest `archive/<slug>/<sha>` tag, the in-flight bound on a slug whose worktree still exists, the repository flag's default that now applies whenever the flag is empty and the foreign `-hub` refusal.
The opening sentence and the repository-flag sentence are reworded to say the report holds time as well as tokens and that the default applies whenever the flag is empty.
Each sentence stays on its own line, per `PATTERN-comment-line-breaks`.
The description comments at the top of `tokencount.sh` and `tokencount.cmd` say the report holds token use and time by role, that the repository flag's default is the current worktree's weft sibling whenever the flag is empty, and that a slug still checked out under the hub reads no archived status.
