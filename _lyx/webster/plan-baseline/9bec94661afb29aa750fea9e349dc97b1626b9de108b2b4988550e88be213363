# Card 16 — verify-gate-fix-commits

**Create:**
- `plan:internal/websterengine#VerifyGateNotes.addCommits` -> `func (n *VerifyGateNotes) addCommits(entries []CommitLogEntry)`
- `plan:internal/websterengine#VerifyGateNotes.FixCommits` -> `func (n *VerifyGateNotes) FixCommits() []CommitLogEntry`
- `plan:internal/websterengine#AppendVerifyGateFixCommits` -> `func AppendVerifyGateFixCommits(websterDir string, commits []CommitLogEntry) error`
- `plan:internal/websterengine#TestVerifyGate_NotesFixCommitsOnce` -> `func TestVerifyGate_NotesFixCommitsOnce(t *testing.T)`

**Edit:**
- `internal/websterengine#VerifyGateNotes`
- `internal/websterengine#VerifyGateNotes.Apply`
- `internal/websterengine#verifyGateSeams`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#newVerifyGate`
- `internal/websterengine/doc.go`
- `internal/websterengine#gateFake`
- `internal/websterengine#gateFake.seams`
- `internal/websterengine_test#TestAppendSummarySections`
- `internal/websterengine_test#TestRun_VerifyGate`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-body-verify-fix.md`

**Uses:**
- `plan:internal/websterengine#CommitLogEntry`
- `internal/websterengine#appendToSummary`

**Intent:**
D2.8: the verify-fix fork's `fix:` commits and `Pattern-Departure:` trailers reach `summary.md` through Go, so the Master still never reads raw fork output.

- `verifyGateSeams` gains `commitLog func(base string) ([]CommitLogEntry, error)`, the non-merge first-parent commits from base to HEAD; `NewVerifyGate` fills it by resolving HEAD through `geom.git().HeadSHA` and calling `geom.git().CommitLog(geom.WorktreeRoot, base, head)`, since card 15's `CommitLog` takes full SHAs.
- `newVerifyGate`, at each evaluation with a recorded pre-fix head, calls it right after `fixRejection` passes, before the verify runs, and hands the entries to `notes.addCommits`, so the passing evaluation after a fix records it too; a clean parent merge in the range is a merge and is neither listed nor read for trailers.
  A `commitLog` error is the gate's error, as `fixRejection`'s is.
- `VerifyGateNotes` gains a commits list under its mutex; `addCommits` keeps each commit once by SHA, in first-seen order, the way `addFlaky` keeps each identity once; `FixCommits` returns the entries whose subject starts with `fix:` or that carry a departure.
- `Apply` loses its early return on an empty flaky list: it writes the flaky identities only when there are some, as today, and calls `AppendVerifyGateFixCommits(websterDir, n.FixCommits())` either way, after Run's wait and so after the Master's last rewrite of `summary.md`; its doc says so.
  `AppendVerifyGateFixCommits` appends nothing when no entry's subject starts with `fix:`; otherwise it appends a `## Verify gate fix commits` section through `appendToSummary`, one bullet per `fix:` commit naming its SHA and subject, then one bullet per departure line (`DepartureLines`) of every listed entry, a non-`fix:` entry's departures included.
- `webster-template-master.md`'s gate step 4 renames its section to `## Verify gate findings`, which names only the findings from the findings file, never what the fixer's reply says, and says Go appends the `## Verify gate fix commits` section after the run; the "never read raw fork output" rule keeps no exception.
- `webster-body-verify-fix.md`'s "Your final action" no longer asks the fixer to name each commit: its reply names only a failure it could not fix, which the Master's outcome rewrite needs; a new rule bullet points a departure from a PATTERN entry at the `Pattern-Departure:` trailer section of `loom-plan-spec.md` for its `fix:` commit.
  The fork body is told no specs directory, so the pointer names the section and the spec by name.
- `doc.go`'s verify gate paragraph names the notes' commits and the appended section.

Tests:
`gateFake` gains `commitLogs`, the entries `gateFake.seams`' `commitLog` returns, `commitLogErr`, the error it returns in their place, and `commitLogCalls`, which it counts; the existing `logs` field keeps the verify-log contents.
`TestVerifyGate_NotesFixCommitsOnce` drives a failing evaluation, then two passing-path evaluations over the same log of a `fix:` commit with a trailer and a non-fix commit: `FixCommits` lists the fix commit once with its departure, and the evaluation before any pre-fix head leaves `commitLogCalls` at zero; a second gate whose `commitLogErr` is set returns that error from the evaluation after its failure.
`TestAppendSummarySections` gains rows: fix commits append the section after the existing summary with each SHA, subject and departure line; a fix commit beside a non-fix commit that carries a departure lists that departure too; entries with no `fix:` subject, one of them carrying a departure, append nothing; a fix commit with no trailer names no departure.
`TestRun_VerifyGate`'s "a failure then a pass ends done" step commits a `fix:` commit carrying a `Pattern-Departure:` trailer in its scratch repository before the passing evaluation, and asserts after the run that `summary.md` holds `## Verify gate fix commits` naming that commit's SHA, subject and departure line, with no flaky identity recorded, so an `Apply` that skips the call or keeps its early return fails the step.

**ImpactSummary:** the gate makes one more read-only git call per evaluation after a failure, and `Apply` gains one summary section; a run that never failed the gate is unchanged.
