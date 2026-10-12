# Card 7 — batten-change-classifier

**Create:**
- `internal/battenshed#StateChange`
- `internal/battenshed#ParseStateChange`
- `internal/battenshed#NoticeRules`
- `internal/battenshed#BattenRecord`
- `internal/battenshed#Wake`
- `internal/battenshed#AutomaticStep`
- `internal/battenshed#Classification`
- `internal/battenshed#ClassifyRecords`
- `//internal/battenshed/classify_test.go`

**Edit:**
- `internal/battenshed#stateChangeText`
- `internal/battenshed#TestInnerRun_EachStateChangeIsItsOwnHistoryEntry`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D6 classifies batten's recorded child state changes in `internal/battenshed`, beside `stateChangeText`, which writes them, applying batten's own notice rules to recorded state rather than to a live child.
The new code lives in `classify.go`, takes only values (no path, no clock, no seam) and keeps the package's import allowlist (`shedengine`, `logger`, `parentlock`) unchanged.

`StateChange` is one recorded change: `From`, `To` (`shedengine.State`), `Producer` (empty when the text names none) and `At`.
`ParseStateChange` is `stateChangeText`'s inverse over an entry's output: `child <from> → <to>` with an optional ` at <producer>` suffix, both states among the six legal values; anything else is false, and `At` is left zero.
`stateChangeText` gains a comment naming `ParseStateChange` as its reader and keeps its output byte for byte.

`NoticeRules` carries the row names batten is told in production, so the classifier names none itself: `VerifyRows` (the rows whose first verify failure batten retries, `Merge-In` and `Publish`), `GateRow` (`PR-Gate`), `EpisodeRows` (batten's own `Land` and `Worktree-Teardown`) and `Escalations`, a map from a Bouncer row to the number of its segment's rounds that hold an escalation file.
`BattenRecord` is one run's batten source: `Snapshots`, the batten `status.json` decoded at each records commit in the run's span that holds it, oldest first, the last being the batten record D4 names; and `Child`, the child's loom history at its archive tag.

`ClassifyRecords` reads the state changes from the last snapshot's history, in order (Run-Shed's budget-exempt entries whose output `ParseStateChange` accepts, `At` taken from the entry's `at`), and returns a `Classification` holding `Changes`, `Wakes`, `Automatic`, `Paused` and `AccountedChild`, the indexes of `Child` entries an automatic step accounts for.
The rules, in D6's terms:
- A change into `paused` is not a wake; it joins `Paused`, unless it is part of a verify retry below.
- A change into `blocked` at a `VerifyRows` row that is the first `blocked` at that row in the run, followed in `Child` by a `goto` entry dated after it and before the next change into `awaiting`, `blocked` or `failed`, is batten's own retry: an `AutomaticStep` of kind `verify-retry`, with that `goto` entry's index in `AccountedChild`.
  The retry's own changes after the `blocked` (the child into `paused` at the goto's target and back into `running`) belong to that step, so they join neither `Paused` nor any wake.
  A later `blocked` at the same row is a wake.
- A change into `awaiting` at `GateRow` whose next `GateRow` entry in `Child` dated after it is a `reroute` is batten's resume for a moved parent: an `AutomaticStep` of kind `parent-moved-resume`, with the reroute's index in `AccountedChild`.
- Every other change into `awaiting`, `blocked` or `failed` is one `Wake`: kind `review-escalation` for the first `Escalations[row]` such changes at a Bouncer row, kind `pr-gate` at `GateRow`, and otherwise kind equal to the new state with `Producer` set.
- A failure episode of an `EpisodeRows` row is one `Wake` of kind `failed` at that row: walking `Snapshots` oldest first, a snapshot in state `failed` at the row opens an episode when none is open, and the episode closes at the first later snapshot whose history holds more `done` entries for the row than the opening snapshot's; an episode with no such snapshot runs to the end of the snapshots.
An empty `Snapshots` returns an empty `Classification`.

Tests:
- `TestInnerRun_EachStateChangeIsItsOwnHistoryEntry` adds that `ParseStateChange` of each entry's output returns the `From`, `To` and `Producer` the loop drove, which pins the reader to the writer.
- `TestClassifyRecords` (new, in `classify_test.go`) is table-driven over hand-built `BattenRecord`s, each row asserting the exact `Wakes`, `Automatic`, `Paused` and `AccountedChild`:
  one Bouncer escalation that halts the child (an `awaiting` at `Plan-Bouncer` with `Escalations` 1) is one `review-escalation` wake;
  an `awaiting` at `Plan-Bouncer` with `Escalations` 0 is one wake of kind `awaiting` at `Plan-Bouncer`;
  an `awaiting` at `PR-Gate` with no later reroute is one `pr-gate` wake;
  an `awaiting` at `PR-Gate` followed by a `PR-Gate` reroute in `Child` is one `parent-moved-resume` step and no wake;
  a first `blocked` at `Publish` followed by a `goto` in `Child` and the changes into `paused` at the target and back into `running` is one `verify-retry` step, no wake and an empty `Paused`;
  a first `blocked` at `Merge-In` with a following `goto`, then a second `blocked` at `Merge-In`, is one `verify-retry` step and one wake of kind `blocked` at `Merge-In`;
  a `blocked` at `Plan-Write` is one wake of kind `blocked` at `Plan-Write`;
  a change into `paused` is in `Paused` and is no wake;
  a `Land` failure episode is one wake;
  a `Worktree-Teardown` episode with two `failed` snapshots before its `done`, followed by a `Land` `failed` snapshot after that `done`, is two wakes, one per row;
  a history entry whose output is no state change is ignored.
  Its last rows feed `ParseStateChange` directly: `child running → blocked` (no producer) parses, `child running → stuck at X` and `batten waiting` do not.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/battenshed` to 110.

**ImpactSummary:** `stateChangeText` gains only a comment and its one caller in `innerrun.go` is unchanged; one existing test gains an assertion.

**Commit:** `7: classify batten's recorded child state changes`
