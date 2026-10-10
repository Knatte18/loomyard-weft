# Card 8 — thin-table-rows

**Edit:**
- `internal/websterengine/runlevel_test.go`
- `internal/shuttleengine/waiting_test.go`

**Uses:**
- `internal/websterengine#autoRebaseline`
- `internal/shuttleengine#payloadShellTask`
- `internal/shuttleengine#transcriptShellTask`

**Intent:**
Item 6 of the board entry: the rebaseline and shell-wait tables gain the rows the landing review called thin, as rows of the tables that already cover each surface.

`TestRun_AutoRebaseline` gains a subtest for an accepted auto-rebaseline where only the Card Index of `00-overview.md` changed, the one-line intent of card 1's index line reworded with no card file touched: the Master death the fixture's `diedMaster` helper scripts opens with the rebaseline warning, which contains "accepting no card file".
Its existing subtest "an in-flight card's edit is recorded unrendered and a recovery renders it" gains one assertion: the warning the error opens with contains "amended in-flight cards: 01-batch1".

`TestWait_RecordsEndedShells` gains a column for the outstanding tasks, defaulting to the transcript-reported shell, a column for the expected `EndedShells`, defaulting to that shell's entry, and two rows:
one on the ungated at-once finish with the transcript-reported shell beside a payload-reported shell of a second id, expecting both in `EndedShells` each with its own signal;
one with an awaited shell alone, which pins "an awaited shell never lands there" where the wait holds it.
An ungated at-once finish counts every outstanding task and records each shell among them, awaited or not, so that row cannot carry the criterion and this card changes no production code.
The awaited row instead runs gated and fresh, under a spec whose `AwaitedShellPrefixes` matches the shell's label, with every output file present: `countsAtOnce` is false for the awaited shell, so the waiting turn end does not count, and a `multiStepClock` step appends a `STOP:` line with nothing outstanding, at which the run finishes `OutcomeDone` through the passing gate with `EndedShells` empty and neither ended-shell log line.
The table gains a column for the run's gate, the started-fresh mark, the awaited-shell prefixes and the clock steps that row needs, each zero on the existing rows, and a `wantNoShells` mark the awaited row sets to expect `EndedShells` empty, since a zero expected-`EndedShells` column means the default entry.
Its `wantLog` and `wantNoLog` columns become lists of lines that must and must not appear, so the awaited row lists no required line and both ended-shell lines as absent.
The existing "an awaited shell keeps waiting past the bound" row of `TestPollEventsTick_OutstandingBackgroundWork` pins the wait and is not repeated.

**ImpactSummary:** test rows and one assertion only.
