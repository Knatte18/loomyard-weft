# Card 6 — time-docs

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
Describe the time the report now carries in the package doc and the launchers' description comments, and correct the weft paragraph cards 4 and 5 made wrong.
Only the package doc comment at the top of `main.go` changes, one sentence per line (`PATTERN-comment-line-breaks`); no code in the file changes.

In the package doc:
- The opening sentence says the report gives token use and time per run and per role.
- The repository paragraph (today the weft flag's default "applies whenever the flag is empty and either no slugs are named or the calibration is asked for") says the default applies whenever the weft flag is empty, that `-hub` naming a directory other than the current worktree's parent with the weft flag empty is refused, naming the weft flag, and that outside a resolvable worktree a run without the weft flag fails, named slugs included.
  The sentence at the top that calls the weft flag "the one repository flag" adds that the archive tags also hold each finished run's shed records.
- A new paragraph after the share column's paragraph covers time, each point stated once:
  - a session's working time is its turn time: of the intervals between its consecutive timestamped user and assistant lines, every one but those after a line that ends the turn, an assistant line with a stop reason other than `tool_use`, a user line starting with `[Request interrupted by user` or with `<local-command-stdout>`; a foreground tool call counts, a crashed session's gap until its next line counts;
  - a role's time is the union of its sessions' working intervals clipped to the run's span, `<role>+sub` the union of its sub-agents';
  - the span runs from the seed's `started_at`, or the run's earliest transcript line when the seed has none, bounded below by the committer time of the slug's next-older archive tag and above by the first history entry's finish, to the latest finish among the history entries, a folded entry finishing at its `last_at`;
  - the archived `seed.json` and `status.json` are read from the newest `archive/<slug>/<sha>` tag by committer time, through `internal/shedrun`'s paths and seed decoder and `gitrepo`'s read side; a run whose worktree `<hub>/<slug>` still exists is in flight and reads no tag, and a run with no tag or no status uses its transcript window as its span, says no shed status was read and has no Producers table; an archived seed or status that does not decode aborts the report, naming the slug, the tag and the file;
  - the `time` and `time share` columns follow `share`; the orch row's are blank; the `idle` row is the span less the union of every one of the run's worktree sessions' working intervals, the orch's sessions and sub-agents never included, so parent review is idle;
  - the per-run Producers table splits the span by history entry, each entry owning the time from the latest earlier finish to its own, and gives per producer the entries, the elapsed time, its idle part and its share of the span, elapsed summing to the span;
  - "All runs" sums each run's role times, idle, spans and producer rows, each computed within its run.
- The paragraph that says where the "Orchestrator" section sits stays true; leave it.

In `tokencount.sh` and `tokencount.cmd`, the first description line says the launcher writes the token and time report, and a line after the argument lines says that named slugs read the current hub's weft too, so `-hub` naming another hub needs the weft flag.
