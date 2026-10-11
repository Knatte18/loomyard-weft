# Card 7 — package doc and launcher descriptions

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:** The record asks that the package doc and the launchers' description comments describe the time columns, the idle row, the Producers table and how each is measured.
Cards 1 to 6 build the measurement and the report, and the next card wires the archived records into `run`; this card writes the docs of the behavior the plan ends with, and its `main.go` edit touches the package doc comment only.
The package doc is the comment above `package main`; it stays one sentence per line, as `PATTERN-comment-line-breaks` requires.
It changes in these places:

- The opening sentence says the report covers each run's time as well as its token use, and names the report title "Token usage and time by role"; the report path `.scratch/token-usage-by-role.md` is unchanged.
- The paragraph on the weft repository flag says the default, the current worktree's weft sibling, applies whenever the flag is empty, and that when the hub flag names a directory other than the current worktree's parent and the weft flag is empty the tool refuses and names the weft flag; the earlier wording that ties the default to no slugs or to the calibration is removed.
- A new paragraph says each run's archived `status.json` and `seed.json` are read from the run's `archive/<slug>/<sha>` tag in that repository through `gitrepo`'s read side, from the tag with the newest commit when a slug has several, and that a named slug whose worktree `<hub>/<slug>` still exists is in flight and reads no tag.
  It says an archived seed or status that fails to decode aborts the report, naming the slug, the tag and the file, and that a run with no tag, or a tag with no `status.json`, has no shed status.
- A new paragraph defines working time: for each transcript, its user and assistant lines with a timestamp, in timestamp order, define intervals between consecutive lines, and an interval is working time unless its first line ends the turn, which an assistant line with a `stop_reason` other than `tool_use`, an interrupt marker and a local command's output each do; a foreground tool call counts as working.
  It says a role's time is the union of its transcripts' working intervals clipped to the run's span, that `<role>+sub` is its own union, and that the orch row's time cells are blank.
- The same paragraph defines the span as running from the seed's `started_at`, or the earliest transcript line when the seed has none, bounded by the commit time of the slug's next-older archive tag and by the first history entry's end, to the latest end among the status history's entries, a folded entry ending at its `last_at`; with no shed status it is the transcript window.
- The same paragraph defines the `idle` row as the span minus the union of the working intervals of the run's worktree transcripts and their sub-agents, so orch sessions are never in it and parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push are idle; it says "All runs" sums each run's role times, idle and span with no union across runs, and that role shares can sum past 100% where sessions overlap.
- A new paragraph defines the Producers table: each history entry owns the interval from the latest end among the entries before it, the span start for the first, to its own end; a row's elapsed is the gap between finish times and includes any wait before the row started; the table gives entries, elapsed, idle and share per producer name, elapsed sums to the span, and `Publish` and `Finalize` appear as one row each.

`tokencount.sh` and `tokencount.cmd` each keep their text and gain description lines, in the comment style of each file (`#` and `REM`), saying the report holds token use and time by role, with time and time share columns beside the weight columns, an `idle` row, and a Producers table of the run's shed rows, and that the shed records are read from the weft's archive tags.
They name the output path as before.
No command line in either launcher changes.

**ImpactSummary:** Comments only in three files; no code path, flag or launcher command changes, and no test asserts the doc text.
