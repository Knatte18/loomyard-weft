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
   An interval is working time unless its first line is an assistant line whose `stop_reason` is set and is not `tool_use`, which ends the turn; the interval after such a line is idle until the next user or assistant line.
   Rationale: drivers, writers and reviewers end their turn while a background job, a gate or a parent review runs, and resume on a typed prompt or a task notification, so turn boundaries separate work from waiting without a threshold.
   A foreground tool call, a blocking gate test included, counts as the session's working time because the session is in its turn.

2. **A role's time is the union of its transcripts' working intervals, clipped to the run's span.**
   Concurrent sessions of one role (several burler reviewers, parallel Webster forks) count once.
   `<role>+sub` is its own row, the union of that role's sub-agent transcripts.
   Rationale: union keeps a role's time at or below the span it ran in; the separate `+sub` row mirrors how tokens are counted, which the board asks for.

3. **The run's span runs from the seed's `started_at` to the end of the last history entry.**
   An entry's end is its `last_at` when the entry is folded (`repeats` > 0), else its `at`.
   A run whose archive seed has no `started_at` starts its span at its earliest transcript line.
   A run with no archived status uses its transcript window (`First`..`Last`) as the span, and its section says no shed status was read, so idle covers only the transcript window and no Producers table is printed.
   Rationale: the history's finish times, the measured fact on the board, tile the run from the first row to the last, waits included, while the transcripts miss waits before the first session and after the last.

4. **Idle is the span minus the union of every transcript's working intervals in the run.**
   It is one `idle` row per role table, beside the role rows, with time and time share and blank token cells.
   The report defines idle as time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push.
   Rationale: the board asks for one line per run of time no session spends working; Go-only work cannot be told apart from waiting with the data the run records.

5. **The Producers table splits the span by history entry.**
   Each history entry owns the interval from the previous entry's end (the span start for the first entry) to its own end.
   Per producer name: entries, elapsed (summed intervals), idle (the part of those intervals no transcript works in), and share (elapsed over the span).
   Elapsed sums to the span.
   The report says a row's elapsed is the gap between finish times, so it includes any wait before the row started.
   Rationale: it answers "the time each producer takes" in the shed's own row names and shows which row holds the idle time, from the status the tool already reads.

6. **No start timestamp is added to shed history entries.**
   Rationale: the status records a finish time per entry, so per-phase timestamps exist and the board's "add them" condition does not hold; a start time would only split each gap into before-start and in-row parts, which the board does not ask for and which no archived run carries.
   `Publish` and `Finalize` record no inner breakdown, and their whole interval appears as their Producers row.

7. **Role tables gain `time` and `time share` columns after `share`.**
   `share` keeps its meaning, the weight share.
   Rows keep their weight order.
   The orch row's time cells are blank.
   Durations print as `time.Duration` rounded to the second.
   Time shares divide by the table's span: the run's span per run, the sum of the runs' spans in "All runs".
   The report states that role shares can sum past 100% where sessions overlap (a `+sub` row overlaps its parent), and that the union of working time plus idle is the span.
   Rationale: the board asks for time beside tokens and a share per role.

8. **The archived status is read from the weft repository through `gitrepo`'s read side.**
   The run's tag is the `archive/<slug>/<sha>` tag whose commit is newest by committer date when a slug has several.
   The weft default (the current worktree's weft sibling) now applies whenever `-weft` is empty, not only with no slugs or with `-calibrate`.
   Rationale: the tags and the records already live there, and the tool already reads plan commits and webster records the same way; a run's time needs its status whether or not slugs are named.

9. **Paths and the seed go through `internal/shedrun`.**
   The archived run directory, `seed.json` and `status.json` are named through `shedrun`, and `seed.json` bytes are decoded by `shedrun`; where its current helpers need an on-disk `*lyxcwd.Location`, `shedrun` gains a helper for bytes read from git or for a run id without alias resolution.
   `status.json` is decoded into `shedengine.Status`.
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

- Turn boundaries by `stop_reason` are the working/idle split; an interrupted turn with no end-of-turn line counts its gap until the next line as working.
- The `+sub` rows take time, as they take tokens, though they overlap their parent.
- The Producers table is in scope as the natural reading of "the time each producer takes", built only from the status the board says to read.
- Go-only work (verify, push, Batchifier) counts as idle, since no session is in a turn; the report says so.
- Archived status only; a named in-flight slug gets the no-status fallback.
- The output path stays; only the title changes.

## Open risks

- A transcript line whose `stop_reason` is null on a turn's last assistant line would extend working time to the next line; the sampled finished run has none, but the format is Claude Code's and can change.
- A session interrupted mid-turn (Esc, crash then resume) counts the gap until its next line as working.
- A slug reused across runs mixes older sessions into its transcripts, as it already does for tokens; clipping to the archived span keeps time inside the counted run.
- Changing the weft default to apply always makes a named-slug run outside a resolvable worktree fail where it ran before.

## Acceptance criteria

- A transcript with two turns separated by an `end_turn` and a long gap counts both turns as the role's time and leaves the gap out.
- A foreground tool call inside a turn counts as working time.
- Two overlapping sessions of one role count their union, not their sum; sub-agent transcripts count under `<role>+sub`.
- With an archived status, the span runs from the seed's `started_at` to the last entry's end, a folded entry ending at its `last_at`; idle is the span minus the union of all working intervals.
- The Producers table's elapsed column sums to the span, and each producer's idle is the part of its intervals no transcript works in.
- A seed without `started_at` starts the span at the earliest transcript line.
- A run with no archive tag keeps its role times, uses the transcript window as its span, prints no Producers table, and says no shed status was read.
- A slug with several archive tags reads the newest tagged commit's status.
- The role tables carry `time` and `time share` columns and an `idle` row; the orch row's time cells are blank; "All runs" sums times, idle and spans across runs.
- The package doc, the report title, and the launchers' description comments describe the time columns, the idle row, the Producers table and how each is measured.

## Notes for the plan writer

- Run timestamps are already parsed in `RunTally.observeTime` and `ForkTally.observe` in `tools/tokencount/count.go`; `countFile` walks every transcript once and is the natural place to collect intervals.
- An archived run's records sit at the tag's tree under the run's shed directory, `status.json` and `seed.json`; `gitrepo.Repo.FileAtRevision` reads a blob at a revision.
- `shedrun.StatusRel` and `SeedRel` take a `*lyxcwd.Location` for alias and legacy-`self` resolution against the disk, which does not fit a tree read from git.
- `finishedRuns` already lists the `archive/` tags.
