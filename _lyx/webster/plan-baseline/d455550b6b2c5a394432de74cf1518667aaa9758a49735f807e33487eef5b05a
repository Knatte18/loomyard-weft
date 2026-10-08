# Card 6 — tokencount measures each Merriam session's start

**Create:**
- `tools/tokencount#MerriamStart`
- `tools/tokencount#readMerriamStart`
- `tools/tokencount#launchPointerLine`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `tools/tokencount/count_test.go`

**Intent:**
D4's measurement: tokencount reads, for each webster-role session of a run, the context at the first point that holds Merriam's whole computed start but the plan overview.

`MerriamStart` is one Merriam session's start:
- `Session`, the session transcript's file name;
- `Started`, the timestamp of its first line that carries one;
- `StartContext`, the input + cache write + cache read of the first assistant message with usage after the line holding the tool result of Merriam's Read of its prompt file, read as a fork's start context is;
- `NoStart`, why no start was measured (no launch line, no Read of the prompt file, or no message after its result), empty when `StartContext` was measured.

`readMerriamStart` scans one session file in order:
- the prompt path comes from the first user message whose text (string content or a text item) matches `launchPointerLine`, the launch line lyx types: `Read <path> in full first; it is your complete, authoritative instructions.`;
- the Read is the first assistant `tool_use` named `Read` whose `file_path` input is that path;
- its result is the user line carrying the `tool_result` for that `tool_use` id;
- the measured message is the first assistant message with usage after that line.
lyx types the role skills' load turn before the launch line, so the skill bodies precede the measurement point and land in it; a session whose first Read is of something else still measures at the prompt's Read.

`RunTally` gains `Merriams []MerriamStart`, one per webster-role session, sorted by `Started`; `CountRun` calls `readMerriamStart` for each webster-role session beside `recordBase`.

Tests: `count_test.go`, in `TestCountRun`: a webster session whose skill-load turn and its assistant messages precede the launch line and the prompt Read measures `StartContext` at the first assistant message after the Read's result, not at an earlier message; a second webster session of the same run is listed second by start time; a session with no prompt Read carries a `NoStart` reason.

**ImpactSummary:** `RunTally` gains one field filled by a new per-session scan; the existing role, fork and base tallies are unchanged.

**Commit:** 6: tokencount: measure each Merriam session's start at its prompt Read
