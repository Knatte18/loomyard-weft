# Card 6 — checklist-arbiter-lines

**Create:**
- `plan:internal/loomcli#checklistArbiter` -> `type checklistArbiter struct`
- `plan:internal/loomcli#readChecklistArbiters` -> `func readChecklistArbiters(arbiterDir string) []checklistArbiter`
- `plan:internal/loomcli#checklistArbiterLine` -> `func checklistArbiterLine(a checklistArbiter, now time.Time) string`

**Edit:**
- `plan:internal/loomcli#checklistInput`
- `internal/loomcli#loomCLI.checklistPoll`
- `internal/loomcli#renderChecklist`
- `plan:internal/loomcli#checklistReviewBlock`
- `internal/loomcli#TestRenderChecklist`

**Intent:** D3: each arbiter attempt shows as one line in the round it decided.

`checklistArbiter` holds what one attempt directory gave: the attempt `Number`, `SpawnedAt`, the stop key's `HistoryLength`, its `Deadline`, the envelope (found or not) and whether reading the envelope failed, or, for an attempt whose record could not be read, only its directory name.
`readChecklistArbiters` lists the subdirectories of the run's arbiter directory and reads each on its own, read-only and without a lock (`loomengine.ReadArbiterAttempt`, then `loomengine.ReadArbiterEnvelope` at `loomengine.ArbiterEnvelopePath`), so one bad attempt leaves the others drawn; an absent directory gives none, an attempt whose record errors gives an unreadable entry naming its directory, and an envelope read error gives `verdict unreadable` on that attempt only.
It reads the envelope, never the agent's `verdict.yaml`, so a hand-on the verb concluded itself shows too.
`checklistPoll` resolves the directory once through `shedrun.ArbiterDir(location, shedrun.SelfRunID)` (PATTERN-told-geometry: the CLI resolves, the renderer is told) and calls `readChecklistArbiters` on every frame, filling a new `arbiters` field of `checklistInput`.

`checklistArbiterLine` renders `<spawn hh:mm> arbiter attempt <n> <verdict>`: the envelope's verdict word followed by ` (<cause>)` when its `Cause` is set; with no envelope, `running` while `now` is before the attempt's `Deadline` and `no verdict` from it on; `verdict unreadable` when the envelope read failed.

Placement in `renderChecklist`, applied to every list the frame draws (card 4's per-row visit lines, card 5's round lines, and the unreadable-recipe history list):
- A readable attempt whose `HistoryLength` is k, 1 <= k <= the history's length, stands right after the line of history entry k (index k-1), at that line's indentation; inside a review block that puts the escalation's awaiting line and the arbiter line together inside the round the Bouncer escalated in, with the round the resumed Bouncer opens after them.
- A readable attempt whose k exceeds the history's length, or is below 1, stands at the end of the frame after the history-only rows.
- An unreadable attempt stands as `arbiter attempt unreadable (<directory name>)` at the end of the frame, after the history-only rows and the end-placed attempts and before the state and wait lines.
- A run with no arbiter directory draws no arbiter line.
Only the arbiter is shown; tackler and alignment attempts are not read.

New rows of `TestRenderChecklist`, built from `checklistArbiter` values so the renderer stays pure:
- A round-3 escalation followed by an arbiter `resume` attempt whose stop key names the awaiting entry: the awaiting line (`escalated`) and the arbiter line inside round 3, round 4 after them.
- An attempt whose envelope carries `hand-on` with cause `died` renders `hand-on (died)`.
- An attempt with no envelope renders `running` before its deadline and `no verdict` after it.
- An attempt whose k exceeds the history renders at the end of the frame.
- An unreadable attempt beside a readable one: the readable one in place, `arbiter attempt unreadable (<dir>)` before the state line, and the frame still drawn.
- No arbiter attempts: no arbiter line.
`readChecklistArbiters` is covered by its own subtest block inside `TestRenderChecklist` (a `t.Run` beside the table loop, not a table row), over a temporary arbiter directory holding one readable attempt with an envelope, one without, one whose `envelope.json` holds undecodable bytes (rendering `verdict unreadable` through `checklistArbiterLine`), and one directory without `attempt.json`, asserting the four entries and that an absent directory gives none.

**ImpactSummary:** Confined to the checklist file and its test; adds read-only reads of the run's arbiter directory to the pane's poll.
