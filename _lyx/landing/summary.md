# tokencount: report wall-clock time per role, idle and producer

`tools/tokencount` now reports where each finished run's wall-clock time goes, beside the tokens it already counted.
Until now the report said nothing about time, so the slowest producer and the biggest wait were invisible.

## What changes

- The role tables, per run and in "All runs", gain `time` and `time share` columns after `share`.
  Sub-agents sit under `<role>+sub`, as for tokens.
  The orch row's time cells stay blank, since orch spans many runs.
- Each role table gets an `idle` row: the part of the run's span in which no session of the run is in a turn.
  Parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push all land there.
- A per-run "Producers" table, and its sum in "All runs", splits the span by shed history entry: entries, elapsed, idle and share per producer name.
  Elapsed sums to the span.
- The report title becomes "Token usage and time by role".
  The output path `.scratch/token-usage-by-role.md` is unchanged.
- The package doc and the descriptions in `tokencount.sh` and `tokencount.cmd` describe the new columns and how each is measured.
- `internal/shedrun` gains run-id-relative paths for `seed.json` and `status.json` and a seed decoder over bytes, so the tool reads archived records from git without naming the `shed` segment or decoding `Seed` itself.

## Design decisions

- **Working time is turn time.**
  An interval between consecutive user and assistant lines is working time unless its first line ends the turn: an assistant line with a `stop_reason` other than `tool_use`, a user line starting `[Request interrupted by user`, or a user line starting `<local-command-stdout>`.
  A foreground tool call, a blocking gate test included, counts as working.
- **A role's time is the union of its transcripts' intervals**, clipped to the run's span, so concurrent sessions count once.
  Shares can therefore sum past 100%, and a `+sub` row overlaps its parent; the report says so.
  "All runs" sums each run's own times, idle and span, with no union across runs.
- **The span** runs from the seed's `started_at` (else the run's earliest transcript line) to the latest end among the status history entries, where a folded entry ends at its `last_at`.
  The start is never earlier than the commit date of the slug's next-older archive tag, and never later than the first entry's end.
  A run with no archived status uses its transcript window, prints no Producers table and says no shed status was read.
- **Idle excludes orch.**
  Orch sessions and their sub-agents enter no union, including an orch sub-agent charged wholly to the run for tokens.
- **No start timestamp is added to shed history entries.**
  Each entry already records when its row finished, so the gaps between entries cover the run, waits included.
  A producer's elapsed is therefore the gap between finish times and includes any wait before the row started.
  `Publish` and `Finalize` record no inner breakdown, so each shows as one Producers row.
- **Status is read from the weft repository** at the newest `archive/<slug>/<sha>` tag by committer date, through `gitrepo`'s read side.
  The weft default now applies whenever `-weft` is empty, which has two bounds:
  - A `-hub` other than the current worktree's parent with `-weft` empty is refused, naming `-weft`.
  - A named slug whose worktree `<hub>/<slug>` still exists is in flight, so no archive tag is read for it and it takes the no-status fallback.
- **Behavior change:** outside a resolvable worktree, a run with `-weft` empty now fails, named slugs included, where named slugs ran before.
- A seed or status that fails to decode, or a run id that fails validation, aborts the report naming the slug, the tag and the file.
- `cmd/lyx/testdata/test-budget.yaml` is raised for the `tools/tokencount` row to fit the new tests.

## To check by hand

- Run `tools/tokencount` against real finished runs and confirm the `idle` row and the Producers table look plausible.
  The unit and integration tests use synthetic transcripts and a hermetic fixture.
- A turn's last assistant line with a null `stop_reason` would extend working time to the next line.
  Sampled runs had none, but the transcript format is Claude Code's and can change.
- A session that crashes mid-turn without an interrupt marker counts its gap to the next line as working.
- A slug reused across runs mixes older sessions into its transcripts, as it already does for tokens.
  The span-start bound and the clip to the span keep the time inside the counted run.
- The last whitespace-only formatting commit and the doc-only `max2`/`min2` commit came after the last `lyx gate test --tags integration ./internal/lyxcwd ./tools/tokencount` run; the full `lyx gate test ./...` and integration tiers are worth running before landing.
