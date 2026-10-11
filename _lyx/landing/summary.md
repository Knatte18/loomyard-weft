# Tokencount default counts only full loom runs

With no slugs named, `tools/tokencount` counted the most recently active finished runs.
When bench pairs had run recently, that picked runs which ran Discussion-Write and stopped, and the report showed a Discussion-only role mix.

The default selection now counts only full loom runs.
A full loom run is a torn-down run whose newest archive tag holds a `seed.json` naming the loom recipe and a `status.json` whose history has a `Finalize` entry.
`-last N` is the N most recent such runs by session activity.
A run stopped partway, a run of another recipe such as darn, or a run whose worktree still exists is skipped.
Slugs named on the command line are still counted as given, so a partial run can be inspected on purpose.

The report header now says "the N most recent full loom runs: …" for the default selection.
When the default skipped more recent runs that are not full loom runs, a second line names them.
With no full loom run found, the error says to name the slugs to count a partial run.

In the code, `ShedRecord` gains the seed's `Recipe`.
The new `fullLoomRun` in `shed.go` makes the decision, and `inFlight` replaces the worktree stat that `run` did inline.
`recentRuns` takes the archive trees and returns the counted and the skipped slugs.
`TestRecentRuns` now covers a full run, a bench pair, a darn run that reached Finalize and an in-flight run.
The package doc and the `tokencount.sh` and `tokencount.cmd` comments describe the new default.
