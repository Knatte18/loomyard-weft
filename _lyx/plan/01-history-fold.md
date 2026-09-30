# Card 1 — history-fold

**Edit:**
- `internal/shedengine#HistoryEntry`

**Create:**
- `plan:internal/shedengine#appendOrFold` -> `func appendOrFold(history []HistoryEntry, entry HistoryEntry) []HistoryEntry`
- `internal/shedengine/historyfold_test.go`

**Uses:**
- `internal/shedengine#episodeStuckCount`

**Intent:**
Add the two fold-recording fields to `HistoryEntry` (D3) and the pure helper that decides between appending and folding (D1, D4), without wiring it into the step yet.

On `HistoryEntry`, add `Repeats int` tagged `json:"repeats,omitempty"` and `LastAt string` tagged `json:"last_at,omitempty"`.
`Repeats` is the number of further identical calls folded into the entry, zero when none;
`LastAt` is the RFC 3339 UTC time of the most recent folded call, empty when none, and `At` keeps the time of the first call.
Extend the type's doc comment to say a budget-exempt Stuck folds into an immediately preceding identical exempt Stuck rather than appending, so one entry can stand for a run of consecutive calls.
Both fields live on `HistoryEntry` itself because `ReadJSONStrict` rejects unknown keys.

`appendOrFold` lives in `internal/shedengine/run.go` beside `episodeStuckCount`.
It always returns a fresh slice and never writes through `history`'s backing array, so the slice read at step 1 stays untouched.
It folds when `entry` is a Stuck with `BudgetExempt` true and the last element of `history` is also a budget-exempt Stuck with the same `Producer`, the same `Output`, and equal `GateAttempts` (both nil, or both non-nil with equal values — compare the pointed-to ints, never the pointers).
A fold returns a copy of `history` whose last element keeps its `At`, has `Repeats` incremented by one and `LastAt` set to `entry.At`.
Every other case — empty history, a counted Stuck on either side, a Done or Awaiting last entry, a different producer, a differing `Output` or `GateAttempts` — returns a copy of `history` with `entry` appended unchanged.
The previous entry's own `Repeats`/`LastAt` never take part in the match, so a folded entry keeps absorbing further identical calls.

`historyfold_test.go` covers, as a table where the shapes allow:

- consecutive exempt Stucks fold: length stays one, `At` is the first call's, `Repeats` counts the folded calls, `LastAt` is the latest call's;
- each append case above appends, and the appended entry carries zero `Repeats` and empty `LastAt`;
- a fold, then a Done (or an Awaiting), then an exempt Stuck appends a fresh entry with no `Repeats`/`LastAt`;
- the input slice and its elements are unchanged after a fold and after an append (compare against a deep copy taken before the call);
- `episodeStuckCount` over a folded history equals its count over the unfolded equivalent, with counted Stucks and a Done mixed in;
- an entry with zero `Repeats` and empty `LastAt` marshals without either key, and a status file written without them decodes under `state.ReadJSONStrict`, mirroring `TestHistoryEntry_MissingBudgetExemptDecodesAndCounts`.

**ImpactSummary:** Two omitempty fields on HistoryEntry keep every existing status file and unfolded entry byte-identical; the new helper has no caller until card 2.
