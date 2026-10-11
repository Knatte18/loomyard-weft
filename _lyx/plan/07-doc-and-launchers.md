# Card 7 — package doc and launchers

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
The Decision 10 and scope text: the package doc in `main.go`, and the description comments of the two launchers, describe the time columns, the idle row, the Producers table and how each is measured.
Only the package comment above `package main` changes in `main.go`; no code does.
The doc's opening sentence says the report holds token use and time per run and per role, each role summed over all runs first, and the report title is named as "Token usage and time by role" with the output path `.scratch/token-usage-by-role.md` unchanged.
The paragraph on the weft repository is corrected: the repository flag defaults to the current worktree's weft sibling whenever it is empty, not only with no slugs or a calibration, and an empty repository flag with a hub flag other than the parent of the current worktree is refused, naming the repository flag.
A new paragraph, one sentence per line as `PATTERN-comment-line-breaks` requires, states how time is measured:
- Working time is turn time: for each transcript, the user and assistant lines with a parseable timestamp, ordered by time, bound intervals; an interval is working unless its first line ends the turn, which an assistant line with a `stop_reason` other than `tool_use`, a user line starting `[Request interrupted by user` and a user line starting `<local-command-stdout>` do, and a foreground tool call counts as working.
- A role's time is the union of its transcripts' intervals clipped to the run's span, a `<role>+sub` row is the union of that role's sub-agent transcripts, and the orch row's time cells are blank.
- The span runs from the seed's `started_at` (else the earliest transcript line), bounded below by the commit date of the slug's next-older archive tag and above by the first history entry's end, to the latest end among the shed history entries, a folded entry ending at its `last_at`; a run with no archived status uses its transcript window, prints no Producers table and says no shed status was read.
- Idle is the span minus the union of every transcript's working intervals in the run, orch sessions excluded, so it holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work.
- The Producers table splits the span by history entry in recorded order, each entry owning the interval from the latest earlier end to its own end, so a row's elapsed includes any wait before it started; its idle is the part no working interval covers, and `Publish` and `Finalize` have no inner breakdown.
- The status is read from the `archive/<slug>/<sha>` tag whose commit is newest, from the weft repository through `gitrepo`, and a named slug whose worktree still exists reads no tag.
The `share` paragraph keeps the weight meaning and adds that `time share` divides by the table's span, the sum of the runs' spans in "All runs", and that role shares can sum past 100% where sessions overlap.
`tokencount.sh` and `tokencount.cmd` keep their commands; their description comments say the launcher writes the token and time report to the same `.scratch/token-usage-by-role.md`.
The comment text is edited with one Edit per site, and the doc is checked with `lyx loom lint-comments`.

**Verify:**
- `lyx loom lint-comments`
