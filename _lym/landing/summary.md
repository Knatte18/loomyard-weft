# lym retro: audit landed runs from their transcripts and propose fixes

`lym retro` turns the hand-set transcript audits under `docs/research/` into one command.
It files a darn task with a fixed, lym-scoped question set, and releases it.
The task's writer runs the retro scripts over the runs' records and transcripts, reads what the figures point at,
and writes a stamped report of proposals, most severe first, under `docs/research/retro/<slug>/`.
Each proposal is routed to a check, a rubric line or a deletion.
The run files nothing on the board; the operator picks which proposals become notes.

## The command

`lym retro [<slug>...] [--since <date>] [--until <date>] [--max-runs <n>]` runs from the hub's prime worktree only (`internal/retrocli`).

- With no slug it covers the runs landed on the parent branch after the last retro and up to now.
  With no earlier retro, it covers the last seven days.
  `--since` and `--until` replace the two ends.
- Named slugs cover exactly those runs, each of which must have landed within the last 60 days.
- The landed runs come from `reportengine`'s landings: a first-parent commit that a finished run claims.
  `reportcli.PrimeGeometry` is exported for this, with no change in behavior.
- It renders the body from the new `retro-template-task` stencil, files a darn task on the `docs` profile with the `research` label,
  and releases it with a second write, as `lym beam merge-parent` does.
- The board write carries the caller's strand name, so a run's agent gets the board's usual refusal of a new task.
  A repository with no `tools/retro` is refused, since the task's writer could not run its scripts there.

`internal/retroengine` holds the logic and reads nothing itself:
the last retro's stamp, the run selection with its cap, the body render and the task's board fields.

## The scripts

`tools/retro/retro.sh <out-dir> <slug>...` is the scripted part.
It is built from the past audits' scripts, which stay as they were in their research folders:
the turn audit's call classes and serial-lookup count, the batching remeasure's refusal and gate-result reading,
and `run-slowness-causes`' status and review-facts readers.
For each run, it reads its records at the newest archive tag and the transcripts of its worktree, forks included.
It writes these JSON lines files:

- `sessions.jsonl`: per session, turns, calls, lookup turns, serial single lookups, guard refusals, and its own gate runs and failures.
  Each session is joined to its usage record for the row, model and cost.
- `refusals.jsonl`: each refused call, with the first words of the refusal.
- `rows.jsonl`: per recipe row, verify gate runs and the reworks handed back to the agent.
- `facts.jsonl` and `keys.jsonl`: every cell of each review perch's rounds table, and the ledger keys that recurred.
- `missing.jsonl`: each run with no archive tag or no transcript directory.

`figures.jq` renders these as `figures.md`.
It shows each producer's median turns and cost, with the sessions at twice either median, and the finding totals per severity and kind column.
The stencil has the writer run `tools/tokencount` and `tools/forkaudit` beside it, as the design asks.

## Design decisions

- **The question set and its scripts.**
  The scripts the figures need now live in `tools/retro/`, beside `tools/tokencount` and `tools/forkaudit`.
  The research folders keep their own copies as records of their runs.
  The question set is in the stencil:
  producers against their medians, serial lookups, refused commands, verify gate failures and fixers,
  recurring findings, instructions with no effect, and missing information.
  The writer reads at most 12 moments.
  A moment is a span of at most 30 turns, or one stencil, heddle passage or `CLAUDE.local.md` section that a span showed an agent acting on.
- **How `lym retro` knows the last retro.**
  It reads the latest stamp: no new record is kept.
  The task slug and the report directory share one name, `retro-<UTC end of the window>`, such as `retro-20261116-140550`.
  The last retro is the newest such name among the report directories in the prime and the entries on the board,
  so a retro still in flight counts, and two retros do not cover the same runs.
- **The cap moves the stamp.**
  Past `--max-runs` (default 10), the oldest runs are kept, and the window ends at the last kept landing.
  The next retro then starts with the first run left out, so nothing is skipped.
  Bound: a retro over named runs is stamped with its newest run's landing.
  The next bare retro starts after that, so a run landed before it and not named is left out.
  Pass `--since` to reach it.
- **Friction-Reflect is kept as it is.**
  It is a per-run step that files public lym issues from notes the agents chose to write.
  The retro is a cross-run audit that files nothing, and the two do not overlap in what they produce.
- **The research index path reaches the stencil as a marker.**
  The stencil citation guard refuses a bare `docs/` path in a stencil body.

## Tests

- `internal/retroengine`: finding the last retro; run selection (after the last retro, the default window, a given range,
  the cap moving the window's end, named runs, and the three refusals); and rendering the shipped stencil with every marker filled.
- `internal/retrocli`: the argument refusals that come before any read.
- `tools/retro` (tier1): `retro.sh` over a fixture records repository and transcript root, checking every table it writes and the figures it renders.
