# Card 6 — package-doc

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:**
Describe the time columns, the idle row, the Producers table and how each is measured (Scope, last acceptance criterion).

The package doc in `main.go` keeps one sentence per line (`PATTERN-comment-line-breaks`):
- The opening paragraph says the report holds token use and time per run and per role.
- A new paragraph after the weight and share paragraph states Decision 1's turn time and its turn-ending lines; Decision 2's union per role, `<role>+sub` its own row; Decision 3's span from the seed's `started_at` (or the earliest transcript line) to the latest history entry end, a folded entry ending at its `last_at`, with both bounds on the start; Decision 4's idle row, orch transcripts never in the union; and that durations print rounded to the second and time shares divide by the span, the summed spans in "All runs".
- A paragraph states the Producers table: each history entry owns the interval from the latest earlier end to its own end, so its elapsed includes any wait before the row started, and the columns sum to the span.
- The repository-flag paragraph (the flag `repositoryFlag` names) says the default applies whenever the flag is empty, that a `-hub` other than the current worktree's parent then needs the flag, that each run's status and seed are read from its newest `archive/<slug>/<sha>` tag, and that a slug whose worktree still exists, or that has no tag, reads none and its section says so.
  The sentence on line 43 saying the default applies only with no slugs or with `-calibrate` is replaced.

`tokencount.sh` and `tokencount.cmd` each describe the report as token usage and time by role in their opening comment; the output path is unchanged.
