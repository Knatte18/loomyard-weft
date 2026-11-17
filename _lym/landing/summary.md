# lym retro: file a darn task that audits landed runs from transcripts

`lym retro` turns the hand-set transcript audits under `docs/research/` into one command.
It files and releases one darn task on the `docs` profile; the task's writer audits the runs and writes a stamped report of proposals, most severe first.
The run files nothing on the board: the operator picks which proposals become notes.

## What changes

- `lym retro [<slug>...] [--since <date>] [--until <date>] [--max-runs <n>]` is a new operator verb, prime worktree only (`internal/retrocli`).
  With no slug it covers the runs landed since the last retro; with no earlier retro it covers the last seven days.
  Named slugs cover exactly those runs, each landed within the last 60 days.
  `--max-runs` caps the runs one retro covers.
  Past the cap the oldest are kept and the next retro starts after them.
- `internal/retroengine` holds the domain logic: finding the last retro, choosing the runs under the cap, and rendering the task's board fields.
- `contracts/stencils/retro/retro-template-task.md` is the task body.
  It orders the writer to run the scripts first, read at most a fixed number of moments the figures point at,
  answer a fixed question set, and route each proposal by the fault it fixes:
  a mechanical fault to a check, a judgement call to a reviewer rubric line, an instruction with no effect to a deletion,
  and a new prompt sentence only as the last resort.
- `tools/retro/` holds the retro's own scripts (`retro.sh`, `transcript.jq`, `figures.jq`, `facts.awk`).
  They read each run's records at its newest archive tag and its Claude Code transcripts, and write `figures.md` and the tables behind it:
  turns and cost per session against medians, serial single lookups, refused commands, verify-gate failures and reworks,
  review rounds and recurring finding keys.
  The task also runs `tools/tokencount` and `tools/forkaudit`.
- `reportcli.PrimeGeometry` is now exported, so `lym retro` reads landings through the same `reportengine` source as `lym report`.
- `docs/overview.md` lists the two packages and the module.

## Design decisions

- **The last retro is the newest stamp, not a record.**
  A retro's slug is `retro-<UTC stamp>` of the end of its window, and so is its report directory.
  `lym retro` takes the newest stamp among the directories under `docs/research/retro` and the retro tasks on the board,
  so a retro filed but not yet reported still moves the next window.
- **The cap never splits a landing second.**
  Runs landed in the same second as the first run left out are all kept or all left out.
  When the cap falls inside the oldest landing's second, that whole second is kept, past the cap.
  The stamp of a capped retro is its last kept landing.
- **Named runs** start their window a second before the oldest landing, so each one landed after it.
  A name with no landing within the lookback is refused, as are more names than the cap.
- **`--until` later than now is refused**, since the stamp would hide runs that land before it from the next retro.
- **A second filing under an existing slug is refused**, naming the two ways out.
- **The filing writes the task, then releases it** with a second board write, because the board gives a new task the draft status.
  When the release fails, the error names the draft task and the `lym board set-status` call that releases it.
- **The board write carries the caller's strand name**, so a run's agent is refused a new task as on `lym board`.
- **Friction-Reflect is kept as it is.**
  It files lym issues from notes the agents chose to write; the retro reads transcripts and writes a report.
- **The scripts are shell and `jq`.**
  They need `bash`, `jq`, `awk` and `git`, and write only under the output directory.
  A run on another agent engine has no Claude Code transcript directory and is listed in `missing.jsonl` instead of failing the figures.

## To check by hand

- No retro has been filed against a real hub, so the task's whole path is unrun: a darn run's start, the writer running the scripts, and a report landing.
  Run `lym retro` on the prime worktree and read the filed task and the figures it produces.
- `Select` in `internal/retroengine/retro.go`, the widening loop where the cap falls inside the oldest landing's second:
  check that it keeps every run of that second and cannot index past the runs.
- `tools/retro/retro_integration_test.go`: check that every fixture path is built from `lymdirs.LymDirName` and none is an inline `.lym` or `_lym` literal.
