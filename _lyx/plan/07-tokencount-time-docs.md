# Card 7 — tokencount-time-docs

**Prosa:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Intent:** The package doc in `main.go` and the description comments in `tokencount.sh` and `tokencount.cmd` describe the time columns, the `idle` row, the Producers table and how each is measured.
The package doc's opening sentence says the report covers time as well as token use, and the title "Token usage and time by role" is named where the report is described.
A new paragraph, one sentence per line, states: working time is turn time, an interval between consecutive user and assistant lines being working unless its first line ends the turn (an assistant `stop_reason` other than `tool_use`, an interrupt marker, a `<local-command-stdout>` line), and a foreground tool call counts as working;
a role's time is the union of its transcripts' working intervals clipped to the run's span, `<role>+sub` its own row, and the `time share` column divides by the span (the sum of spans in "All runs"), so role shares can sum past 100%;
the span runs from the seed's `started_at` to the latest history-entry end, with the fallbacks and bounds for a seed without a stamp and for a run with no archived status;
`idle` is the span minus the union of every worktree transcript's working intervals, the orch session and its sub-agents staying out, and holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work;
the Producers table splits the span by history entry, its elapsed being the gap between finish times, and "All runs" sums it by producer.
The paragraph on the weft flag says its default applies whenever the flag is empty, that the archived status is read from the run's newest `archive/<slug>/<sha>` tag and not for a slug whose worktree still exists, and that a `-hub` other than the current worktree's parent with that flag empty is refused.
The two launcher headers each gain a sentence that the report carries per-role time, the idle row and the per-run Producers table beside the tokens, with the measures defined in the package doc.
Comment lines keep one sentence per line, as `lyx loom lint-comments` checks.

**Verify:** go build ./tools/tokencount
