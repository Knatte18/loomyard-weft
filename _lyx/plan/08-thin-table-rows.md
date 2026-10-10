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

`TestWait_RecordsEndedShells` gains a column for the outstanding tasks, defaulting to the transcript-reported shell, and a column for the spec's awaited-shell prefixes, and two rows on the ungated at-once finish:
one with the transcript-reported shell beside a payload-reported shell of a second id, expecting both in `EndedShells` each with its own signal;
one with an awaited shell alone, under a spec whose `AwaitedShellPrefixes` matches its label, expecting `EndedShells` empty.
The existing "an awaited shell keeps waiting past the bound" row of `TestPollEventsTick_OutstandingBackgroundWork` pins the wait and is not repeated.

**ImpactSummary:** test rows and one assertion only.
