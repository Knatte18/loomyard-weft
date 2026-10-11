# Card 1 — Tokencount: an interval into a turn-start line, or over the stall cap, is idle

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#turnPoint`
- `tools/tokencount#workingIntervals`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount/main.go`
- `tools/tokencount/count_test.go`

**Create:**
- `plan:tools/tokencount#stallCap` -> `const stallCap = 30 * time.Minute`
- `plan:tools/tokencount#startsTurn` -> `func startsTurn(l line) bool`

**Intent:**
Decision D1.
After a crash and `--resume`, the gap between a line that does not end the turn and the resume's prompt counts as working time today, and so does a stall of hours with no prompt at all, such as a suspended host.

`line` gains the transcript's `isCompactSummary` field as `IsCompactSummary bool`.
`turnPoint` gains `Starts bool`, whether the line starts a turn.
`startsTurn` reports a turn-start line: a `user` line with a message whose content holds text (a JSON string, or an item of type `text`) and no `tool_result` item, other than a line `endsTurn` already reports (an interrupt marker or a `<local-command-stdout>` output) and other than a line whose `IsCompactSummary` is true.
`startsTurn` reads the content's item types itself, since `userTexts` drops the `tool_result` items it needs to see.
`countFile` fills `Starts` from `startsTurn` beside `Ends`.

`workingIntervals` keeps the interval from each point to the next, in timestamp order, only when the earlier point does not end the turn (as today), the later point does not start a turn, and the interval is not longer than `stallCap`.
Such an interval counts as idle whole, never clipped to the cap.
`stallCap` is the one named constant, 30 minutes, three times the harness's ten-minute cap on a foreground Bash call; its comment says so.
`endsTurn` is unchanged: a null `stop_reason` still never ends a turn, since real transcripts write it mid-turn.

What this lets through and its bound: the turn-start rule removes only an interval whose later line is a turn-start line, and a mid-turn user text line (a skill body after a `Skill` result, a queued operator message) loses only the short gap into it.
The cap removes a single interval over 30 minutes whole, so a foreground call that blocks longer loses that interval from the waiting session while the sub-agent's own transcript still counts its work.
A stall under the cap that ends at a line other than a turn-start line still counts as working.

`Report.WriteMarkdown`'s explanatory line after "Time is turn time:" gains one sentence saying an interval into a line that starts a turn, or longer than the stall cap, is idle, with the cap rendered from `stallCap`.
The package doc in `main.go`, in the paragraph that defines turn time, states the turn-start line (and the three exclusions), the stall cap and that either makes the interval idle whole, one sentence per line.
`tokencount.sh` and `tokencount.cmd` describe no measurement rule, so they stay as they are.

Coverage extends `TestRunTimeByRole` in `count_test.go`, with no new top-level test (the package sits at its budget row), using fixed timestamps:
- an assistant `tool_use` line followed an hour later by a user prompt line, and a `tool_result` line followed an hour later by a prompt line: that hour is idle for the role and the run;
- a `tool_use` line followed 32 hours later by its `tool_result`, and a `tool_result` line followed an hour later by an assistant line: idle;
- a `tool_use` line followed 20 minutes later by its `tool_result`: working;
- an interval under the cap ending at an interrupt-marker line: working;
- a user text line one second after a `tool_result`, then an assistant line: only that one-second gap is dropped;
- a `/compact` command line followed five minutes later by a user line with `isCompactSummary` true: working.
The existing normal-turn assertions (prompt, `tool_use`, `tool_result`, `end_turn`) keep their exact figures, and the existing burler fixture's figures are unchanged, since none of its intervals ends at a turn-start line after a non-ending line or exceeds the cap.

**ImpactSummary:** confined to `tools/tokencount`'s working-interval computation and report text; role time, idle and the Producers table's idle column change only for transcripts with a crash gap or a stall over 30 minutes.

**Commit:** 1: tokencount: a gap into a turn-start line or over 30 minutes is idle
