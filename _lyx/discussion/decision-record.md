# Decision record — run-time-by-role

## Goal

`tools/tokencount` reports where each finished run's wall-clock time goes: the time each role's sessions spend working, the time no session works, and the time between the run's shed rows, each as a share of the run's span, so the slowest producer and the biggest wait stand out.

## Scope

In:

- Per-role working time and time share, per run and in "All runs", as new columns of the existing role tables, with sub-agents under `<role>+sub` exactly as tokens count them.
- An `idle` row per role table: the part of the run's span in which no session of the run is in a turn.
- A per-run "Producers" table, and its sum over runs in "All runs", built from the run's archived shed `status.json` history.
- Reading each run's archived `status.json` and `seed.json` from the run's `archive/<slug>/<sha>` tag in the weft repository.
- The package doc in `tools/tokencount/main.go`, the report title, and the description comments in `tokencount.sh` and `tokencount.cmd`.

Out:

- Any change to `internal/shedengine`'s status shape: no start timestamp is added to history entries (see Decisions).
- Time for the orch row: orch spans many runs, so its time cells stay blank.
- In-flight runs' live status: only archived status is read, matching the tool's finished-run model.
- The calibration and profile sections.

## Decisions

1. **Working time is turn time.**
   For each transcript, the user and assistant lines with a parseable timestamp, ordered by timestamp, define intervals between consecutive lines.
   An interval is working time unless its first line ends the turn: an assistant line whose `stop_reason` is set and is not `tool_use`, a user line whose text starts with Claude Code's interrupt marker, `[Request interrupted by user`, or a user line whose text starts with `<local-command-stdout>`, the output of a local slash command, which starts no model turn.
   The interval after a turn-ending line is idle until the next user or assistant line.
   Rationale: drivers, writers and reviewers end their turn while a background job, a gate or a parent review runs, and resume on a typed prompt or a task notification, so turn boundaries separate work from waiting without a threshold.
   A foreground tool call, a blocking gate test included, counts as the session's working time because the session is in its turn.

2. **A role's time is the union of its transcripts' working intervals, clipped to the run's span.**
   Concurrent sessions of one role (several burler reviewers, parallel Webster forks) count once.
   `<role>+sub` is its own row, the union of that role's sub-agent transcripts.
   Rationale: union keeps a role's time at or below the span it ran in; the separate `+sub` row mirrors how tokens are counted, which the board asks for.

3. **The run's span runs from its start to the latest end among its history entries.**
   An entry's end is its `last_at` when the entry is folded (`repeats` > 0), else its `at`.
   The start is the seed's `started_at`; a seed without one, the usual case for runs archived before seeds were stamped, starts the span at the run's earliest transcript line.
   Either start is bounded: never earlier than the commit date of the slug's next-older archive tag, so a reused slug's older sessions cannot set it, and never later than the first history entry's end.
   A run with no archived status uses its transcript window (`First`..`Last`) as the span, and its section says no shed status was read, so idle covers only the transcript window and no Producers table is printed.
   Rationale: the history's finish times, the measured fact on the board, tile the run from the first row to the last, waits included, while the transcripts miss waits before the first session and after the last.

4. **Idle is the span minus the union of every transcript's working intervals in the run.**
   The union covers the run's worktree transcripts and their sub-agents only.
   The orch session and every orch sub-agent stay out, including one charged wholly to the run for tokens, so parent review counts as idle.
   It is one `idle` row per role table, beside the role rows, with time and time share and blank token cells.
   The report defines idle as time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push.
   Rationale: the board asks for one line per run of time no session spends working; Go-only work cannot be told apart from waiting with the data the run records.

5. **The Producers table splits the span by history entry.**
   Each history entry, in recorded order, owns the interval from the latest end among the entries before it (the span start for the first entry) to its own end, empty when its own end is not later.
   Per producer name: entries, elapsed (summed intervals), idle (the part of those intervals no working interval of the run covers), and share (elapsed over the span).
   Elapsed sums to the span.
   The report says a row's elapsed is the gap between finish times, so it includes any wait before the row started.
   Rationale: it answers "the time each producer takes" in the shed's own row names and shows which row holds the idle time, from the status the tool already reads.

6. **No start timestamp is added to shed history entries.**
   Rationale: history entries already record a finish time per row, so per-phase timestamps exist and the board's "add them" condition does not hold.
   `Publish` and `Finalize` record no inner breakdown, and their whole interval appears as their Producers row.

7. **Role tables gain `time` and `time share` columns after `share`.**
   `share` keeps its meaning, the weight share.
   Rows keep their weight order.
   The orch row's time cells are blank.
   Durations print as `time.Duration` rounded to the second.
   Time shares divide by the table's span: the run's span per run, the sum of the runs' spans in "All runs".
   "All runs" sums each run's role times, idle and span, each computed within its run; no union is taken across runs.
   The report states that role shares can sum past 100% where sessions overlap (a `+sub` row overlaps its parent), and that the union of working time plus idle is the span.
   Rationale: the board asks for time beside tokens and a share per role.

8. **The archived status is read from the weft repository through `gitrepo`'s read side.**
   The run's tag is the `archive/<slug>/<sha>` tag whose commit is newest by committer date when a slug has several.
   The weft default (the current worktree's weft sibling) now applies whenever `-weft` is empty, not only with no slugs or with `-calibrate`.
   That default reads the current hub's weft, which lets a slug be read against the wrong hub's records when `-hub` names another hub.
   The bound: when `-hub` names a directory other than the current worktree's parent and `-weft` is empty, the tool refuses and names `-weft`; both directories are compared as absolute, cleaned paths.
   Reading status for named slugs also lets an in-flight run that reuses an older run's slug take the older run's archive tag as its status.
   The bound: a named slug whose worktree `<hub>/<slug>` still exists is in flight, since teardown removes the worktree, so no archive tag is read for it and it takes Decision 3's no-status fallback.
   Outside a resolvable worktree the default cannot be derived, so a run with `-weft` empty fails there, named slugs included, where named slugs ran before.
   Rationale: the tags and the records already live there, and the tool already reads plan commits and webster records the same way; a run's time needs its status whether or not slugs are named.

9. **Paths and the seed go through `internal/shedrun`.**
   For a tree read from git, `shedrun` gains a run-id-relative path pair for `seed.json` and `status.json` and a decoder over seed bytes with the same strict decoding as its on-disk reader.
   The path pair validates the run id with `ValidateRunID` and applies no alias and no legacy `self` fallback; it joins the same `shed` segment declaration the on-disk paths use.
   `status.json` is decoded into `shedengine.Status`.
   A run id that fails validation, or a seed or status that fails to decode, aborts the report with an error naming the slug, the tag and the file.
   Rationale: `PATTERN-shed-run-directory` makes `shedrun` the sole declarer of the `shed` segment and `seed.json` and the sole decoder of `Seed`; `PATTERN-lyxdirs-single-declarer` keeps `_lyx` out of the tool.

10. **The report title becomes "Token usage and time by role"; the output path `.scratch/token-usage-by-role.md` stays.**
    Rationale: the title names what the report holds; keeping the path spares the launchers and the operator's habit a rename the board did not ask for.

## Constraints

- `PATTERN-shed-run-directory`: no file outside `internal/shedrun` names the `shed` segment or `seed.json` in a path, or decodes `Seed`.
- `PATTERN-lyxdirs-single-declarer`: the tool spells no `_lyx` literal.
- `PATTERN-fabric-git`: the weft is only read, through `gitrepo`'s read side; reads are exempt, and `tools/` is outside the `lyx` binary but follows PATTERN.
- `PATTERN-gitexec-checked-call`: any new git call uses `gitexec.Run`.
- `PATTERN-cwd-resolution`: the tool resolves its cwd only through `internal/lyxcwd`, as it does now.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-testkit`: unit-tier tests over synthetic transcripts and a hermetic weft fixture where one is needed; extend existing tests where they cover the surface.
- `PATTERN-comment-line-breaks`: the package doc keeps one sentence per line.
- Test budget: `cmd/lyx/testdata/test-budget.yaml` has a `tools/tokencount` row; a card adding top-level tests that push the package past it lists that file under `Edit:`.
- cgo build, per `PATTERN-quarry-cgo`.

## Auto-mode assumptions

- A session that crashes mid-turn, leaving no interrupt marker, counts its gap until its next line as working.
- The Producers table is in scope as the natural reading of "the time each producer takes", built only from the status the board says to read.

## Open risks

- A transcript line whose `stop_reason` is null on a turn's last assistant line would extend working time to the next line; the sampled finished run has none, but the format is Claude Code's and can change.
- A slug reused across runs mixes older sessions into its transcripts, as it already does for tokens; the span start's bound at the older tag and the clip to the span keep time inside the counted run.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - named-inflight-slug-reads-older-archive (class decision, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A transcript with two turns separated by an `end_turn` and a long gap counts both turns as the role's time and leaves the gap out.
- A user line holding the interrupt marker ends the turn: the gap after it is idle.
- A local slash command typed between turns, its output a `<local-command-stdout>` user line, adds no working time after that line.
- A foreground tool call inside a turn counts as working time.
- Two overlapping sessions of one role count their union, not their sum; sub-agent transcripts count under `<role>+sub`.
- Orch transcripts, including an orch sub-agent charged wholly to the run, enter no union, so the run's idle holds their time.
- With an archived status, the span runs from the seed's `started_at` to the latest entry end, a folded entry ending at its `last_at`; idle is the span minus the union of all working intervals.
- The Producers table's elapsed column sums to the span, entries out of order included, and each producer's idle is the part of its intervals no transcript works in.
- A seed without `started_at` starts the span at the earliest transcript line, never earlier than the slug's next-older archive tag and never later than the first history entry's end.
- A run with no archive tag keeps its role times, uses the transcript window as its span, prints no Producers table, and says no shed status was read.
- A slug with several archive tags reads the newest tagged commit's status.
- An archived seed or status that fails to decode aborts the report, naming the slug, the tag and the file.
- A `-hub` other than the current worktree's parent with `-weft` empty is refused, naming `-weft`; the same hub spelled with a trailing slash or as a relative path is not refused.
- A named slug whose worktree still exists reads no archive tag, even when an older run's tag for that slug exists, and its section says no shed status was read.
- The role tables carry `time` and `time share` columns and an `idle` row; the orch row's time cells are blank; "All runs" sums times, idle and spans across runs.
- The package doc, the report title, and the launchers' description comments describe the time columns, the idle row, the Producers table and how each is measured.

## Notes for the plan writer

- Run timestamps are already parsed in `RunTally.observeTime` and `ForkTally.observe` in `tools/tokencount/count.go`; `countFile` walks every transcript once and is the natural place to collect intervals.
- An archived run's records sit at the tag's tree under the run's shed directory, `status.json` and `seed.json`; `gitrepo.Repo.FileAtRevision` reads a blob at a revision.
- `shedrun.StatusRel` and `SeedRel` take a `*lyxcwd.Location` for alias and legacy-`self` resolution against the disk, which does not fit a tree read from git.
- `finishedRuns` already lists the `archive/` tags.
