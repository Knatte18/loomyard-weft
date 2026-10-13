# Card 6 — describe-time

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
Describe the time figures where the tool is documented (Scope's last bullet, Decision 10).

`main.go`'s package doc, one sentence per line (`PATTERN-comment-line-breaks`):
- Its opening says the report holds token use and time per run and per role.
- The sentences on the repository flag (`repositoryFlag`) say its default applies whenever the flag is empty, that a `-hub` other than the current worktree's parent with that flag empty is refused, and that outside a resolvable worktree a run with that flag empty fails.
- A new paragraph after the share-column paragraph describes the time columns: working time is turn time, an interval between consecutive timestamped user or assistant lines unless its first line ends the turn (an assistant line stopping other than on `tool_use`, an interrupt marker, a local command's output); a role's time is the union of its transcripts' working intervals clipped to the run's span, `+sub` its own row; time share divides by the span, the sum of spans in "All runs"; the orch row's time cells are blank.
- It describes the span: from the archived seed's `started_at`, or the earliest transcript line, bounded below by the slug's next-older archive tag and above by the first history entry's end, to the latest history entry end (a folded entry's `last_at`); a run with no archive tag, or whose worktree still exists, uses its transcript window and says no shed status was read.
- It describes the `idle` row (time no session of the run is in a turn, orch included in idle) and the Producers table (each history entry owns the time since the latest earlier end, so a row's elapsed includes any wait before it started; elapsed sums to the span), and that the status and seed are read from the newest `archive/<slug>/<sha>` tag's commit in the weft through `gitrepo`'s read side, a decode failure aborting the report.
- The report title it names is "Token usage and time by role".

`tokencount.sh` and `tokencount.cmd`: the opening description comment says the launcher writes the token usage and time report, still to `.scratch/token-usage-by-role.md`.
