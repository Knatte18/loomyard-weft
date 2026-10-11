# tokencount: default selection counts only full loom runs

With no slugs, `tools/tokencount` took the `-last` most recently active finished runs, whatever they were.
On 2026-10-11 that picked only Discussion bench pairs, which stop after Discussion-Write, so the report showed a Discussion-only role mix with no Plan, Webster or review rows.

The default now counts only full loom runs, and `-last N` means the N most recent of them.
A run is a full loom run when its worktree is torn down and the seed and status at its newest archive tag name the loom recipe and hold a `Finalize` history entry.
In-flight runs, runs of another recipe and runs stopped partway are skipped.
Slugs named on the command line are counted as given, so a partial run can still be inspected on purpose.

The report header now reads `for the N most recent full loom runs: a, b.`.
When the default skipped runs more recent than the ones counted, a second line names them: `Skipped N more recent runs that are not full loom runs: x, y.`
With no full loom run at all, the error says so and points at naming slugs.

## Design decisions

- `recentRuns` walks candidates newest session first and returns the counted and the skipped slugs, so the skipped line lists only runs more recent than the last one counted.
- The test lives in `fullLoomRun` (`shed.go`), built on `readShedRecord`, which now also keeps the seed's recipe in `ShedRecord.Recipe`.
- The in-flight check moved from `run` into a shared `inFlight` helper that `fullLoomRun` reuses; behavior is unchanged.
- "Reached Finalize" is the criterion, read from the archived shed history, so it needs no landing information.
- The package doc comment and the `tokencount.sh` / `tokencount.cmd` comments are updated to match.

## Tests

`TestRecentRuns` covers the counted and skipped lists, including a bench pair, a run of another recipe and an in-flight run.
The integration test `TestRunReadsArchivedShedStatus` pins the header lines for named and default selection, and the no-full-run error, against a real git repository.
`lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run as part of this description.
