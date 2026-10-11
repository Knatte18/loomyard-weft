# Card 2 — Tokencount: a run counts only transcripts that start after the slug's previous run

**Edit:**
- `tools/tokencount#CountRun` -> `func CountRun(dir, slug string, bound time.Time) (RunTally, error)`
- `tools/tokencount#RunTally`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#run`
- `tools/tokencount/count_test.go`
- `tools/tokencount/orch_test.go`
- `tools/tokencount/time_integration_test.go`

**Create:**
- `plan:tools/tokencount#transcriptStart` -> `func transcriptStart(path string) (time.Time, bool, error)`
- `plan:tools/tokencount#runBound` -> `func runBound(tags []ArchiveTag, inFlight bool) time.Time`

**Uses:**
- `tools/tokencount#finishedRuns`
- `tools/tokencount#ArchiveTag`
- `tools/tokencount#ShedRecord`

**Intent:**
Decision D2.
A slug's earlier runs share its worktree path, so they share its Claude Code project directory, and a reused slug's report today counts every earlier run's transcripts as its own.

`runBound` returns a run's lower bound from the slug's archive tags, newest first as `finishedRuns` orders them: for an in-flight run (its worktree `<hub>/<slug>` exists) the newest tag's `Committed`, for a finished run the next-older tag's (`tags[1]`, the one `ShedRecord.Older` already reads), and the zero time when there is no such tag.

`transcriptStart` returns the earliest timestamp of a user or assistant line in the transcript at path, read through `lineTime`, and false when no such line carries one.

`CountRun` takes the bound.
With a zero bound it counts every transcript, as today.
With a bound, a main-session transcript whose `transcriptStart` is before the bound, or which has none, is left out whole, with its sub-agent transcripts: it adds nothing to tokens, cost, working intervals, the session count, forks, Merriam starts, the begin-batch base, or the run's `First`/`Last` window, which `CountOrch` reads for the orch charge.
The check runs before `latestTitle`, `countFile`, `recordBase` or `readMerriamStart` reads the session.
A run whose sessions are all left out returns an empty tally, not an error; the "no sessions" error stays for a directory with no transcript at all.
`RunTally` gains `Bound time.Time` (the bound applied, zero for none) and `LeftOut int` (main-session transcripts left out).

`run` in `main.go` reads whether the worktree exists before `CountRun`, which today happens after it, computes `runBound(finished[slug], inFlight)` and passes it in; the shed-record branch keeps reading the same `os.Stat` result.
No new git call site: the tags come from the `finishedRuns` read `run` already makes (`PATTERN-gitexec-checked-call`, `PATTERN-gogit-read-helper`).

`Report.WriteMarkdown`'s per-run section prints, when `LeftOut` is above zero, one line right after the role table stating how many session transcripts were left out and the bound's date in RFC 3339, as transcripts that started before the slug's previous run ended; a run with `LeftOut` zero prints no such line.
The package doc in `main.go` gains the bound rule, one sentence per line, next to the paragraph on which runs count: what the bound is for an in-flight and a finished run, that a transcript is judged by its first user or assistant line, that one with none is left out when a bound exists, everything a left-out transcript is kept out of, and the report line.
No flag disables the filter.

What this can let through and its bound: the filter drops only transcripts that started before the previous run's final tagged commit; a run's own sessions start after its worktree exists, which is after that commit.
A session the previous run started after its final tagged commit is still counted, the decision record's open risk.

`CountRun`'s callers in `count_test.go` and `orch_test.go` pass the zero time, keeping their figures.
Coverage, with no new top-level test (the package sits at its budget row):
- `TestCountRun` in `count_test.go` gains a bounded case: a project directory holding a webster session that starts before the bound, with a begin-batch result, a fork transcript and a launch line, a session with no timestamped user or assistant line, and a session starting after the bound; the tally counts only the later session's usage and sessions, holds no fork, Merriam start or `BaseSHA` from the earlier one, its `First` is the later session's first line, `LeftOut` is 2, and the rendered per-run section carries the left-out line with the bound's date.
- `TestRunReadsArchivedShedStatus` in `time_integration_test.go` drives `run` end to end: the finished `done` slug with two tags gains a transcript starting before the older tag's date, left out, beside its counted one; the in-flight `live` slug's counted transcript moves after its newest tag's date and it gains one starting before that date, left out; a `solo` slug with one tag and no worktree counts its transcript and prints no left-out line; each per-run section's left-out line, or its absence, is asserted.

**ImpactSummary:** `CountRun` is re-signed and called only from `run` and the package's own tests; reports regenerated for reused slugs change figures, and single-run slugs are unchanged.

**Commit:** 2: tokencount: a reused slug counts only its own run's transcripts
