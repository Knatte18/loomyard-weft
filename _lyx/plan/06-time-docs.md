# Card 6 — Package doc and launchers describe the time report

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
Bring the package doc and the launchers' description comments in line with the time report cards 2 to 5 built (decision record, Scope).

In the package doc in `main.go`:
- the opening sentence says the report holds token use and time per run and per role;
- the sentences on the weft flag, the one `repositoryFlag` names, say its default applies whenever the flag is empty, and that a hub flag naming a directory other than the current worktree's parent needs the weft flag;
- new paragraphs describe the `time` and `time share` columns as turn time (a session works from a user or assistant line until the assistant line whose `stop_reason` is set and not `tool_use`, an interrupt marker or a `<local-command-stdout>` line), a role's time as the union of its transcripts' working intervals clipped to the span with sub-agents under `<role>+sub`, the `idle` row as the span minus the union of every working interval of the run with orch sessions left out, the orch row's blank time cells, and "All runs" as sums of per-run figures;
- the span: from the archived seed's `started_at`, or the run's earliest transcript line, bounded by the next-older archive tag's commit date and the first history entry's end, to the latest history entry's end, read from the newest `archive/<slug>/<sha>` tag; a run whose worktree still exists, or with no tag or no status at its tag, uses its transcript window and prints no Producers table;
- the Producers table: each history entry owns the interval from the latest end before it to its own end, so elapsed sums to the span and holds any wait before a row started.

Each sentence stays on its own comment line (`PATTERN-comment-line-breaks`).
In `tokencount.sh` and `tokencount.cmd`, the first description line says the launcher writes the token and time report.

**ImpactSummary:** Comments only: the package doc and the two launchers' header comments; no code changes.
