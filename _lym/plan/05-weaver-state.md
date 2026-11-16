# Card 5 — weaver-state

**Create:**
- `plan:internal/weaverengine#State` -> `type State struct`
- `plan:internal/weaverengine#AttemptState` -> `type AttemptState struct`
- `plan:internal/weaverengine#FixerState` -> `type FixerState struct`
- `plan:internal/weaverengine#StopRecord` -> `type StopRecord struct`
- `plan:internal/weaverengine#LoadState` -> `func LoadState(runDir, scratchDir string) (*State, error)`
- `plan:internal/weaverengine#SaveState` -> `func SaveState(runDir, scratchDir string, st *State) error`
- `plan:internal/weaverengine#StateRel` -> `func StateRel() string`
- `plan:internal/weaverengine#BegunBatches` -> `func BegunBatches(st *State) bool`
- `plan:internal/weaverengine#TestState` -> `func TestState(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#State`
- `internal/gatefixer#Record`
- `internal/websterengine#LoadState`

**Intent:** Declare Weaver's state file, `_lym/weaver/state.json` (D9, D12).

- `State` embeds `runplan.State` and adds:
  `RunGUID`; `PlanWeftCommit`, the weft commit holding the plan that the first begin records (D2, D3);
  `Attempts []*AttemptState`, in attempt order;
  `BatchEnds map[int]string`, each ended batch's end commit, the task branch's HEAD when the batch's attempt ended, done or failed;
  `LastEnd`, the end commit of the batch whose attempt ended last, which D1's repair condition compares with the task branch's HEAD;
  `Fixers []*FixerState`; `Stop *StopRecord`, the row's last batch, needs-parent or verify stop.
- `AttemptState` holds `Batch`, `Number` (the attempt number, one up per attempt of the batch: a same-rung retry and a re-entry attempt included), `Rung`, `ReportPath` (the strand's identity, fixed before the spawn), `StrandGUID` and `StrandName` (empty until the spawn returns or an attach reads them from shuttle's run record), `InFlight`, `Outcome` (empty while in flight, else `done`, `failed`, `lost`, `needs_parent` or `fault`), `Cause`, `Uncommitted` (the tracked paths the attempt left uncommitted, which the reset counts as the run's own), `Delivered` (the decision headings sent to its strand, D13) and `BeganAt`.
- `FixerState` holds `Number`, `ReplyPath` (`fix-<n>.md` in the reports directory), `Rung`, `Record *gatefixer.Record` (nil until `gatefixer` saves it), `StrandName`, `Outcome`, `Findings` and `Delivered`.
- `StopRecord` holds `Kind` (`batch`, `needs_parent` or `verify`), `Batch` and `Rung`, which D7's re-entry reads.
  A plan-conflict Stuck and a fault in Weaver's own config or state record none.
- `LoadState` and `SaveState` read and write through `internal/state` with the lock at `<scratchDir>/state.json.lock`, as `websterengine`'s do: a missing file is `(nil, nil)`, a malformed one a wrapped error naming the path, and an unknown key is kept, never refused (PATTERN-tolerant-records).
- `StateRel` is the state file's anchor-relative path, `_lym/weaver/state.json`, the path the committed-history read takes (card 6).
- `BegunBatches` reports whether a non-nil state holds a batch record.

`TestState` is tier0: a save and load round-trip keeps every field; a file with an unknown key loads; a missing file loads nil; a malformed file errors naming the path; `BegunBatches` over nil, an empty and a begun state.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
