# Card 4 — Record each fork's start context and Merriam session

**Edit:**
- `tools/tokencount/count.go`
- `tools/tokencount/count_test.go`
- `tools/tokencount/main.go`

**Intent:**
Give calibration the measurement D9 fits on.
`ForkTally` gains `StartContext int`, the input + cache write + cache read of the fork's first counted message (the context Merriam held when it spawned the fork), and `Session string`, the file name of the webster-role session transcript the fork's sub-agent transcript sits under.
`countFile` sets `StartContext` on the first message it counts for the fork; a fork with no counted message keeps 0.
The "Webster forks" table gains a start-context column after cards.
`main.go`'s package comment defines both measurements where it defines a fork's peak context.

Tests: `TestCountRun` asserts `StartContext` and `Session` of its fixture forks, including two forks under two different webster sessions, and the new table column.

**ImpactSummary:** two new ForkTally fields and one report column in tokencount; nothing else reads ForkTally yet.
