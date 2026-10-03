# Card 9 — bouncer-env-slug-budget

**Edit:**
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_bouncer.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`

**Uses:**
- `internal/loomrecipe/loomrecipe.go`
- `internal/shedengine/progress.go`

**Intent:** Tell every Bouncer row the slug and the live bounce budget its circling Reason needs (card 8).

`shedrecipe.Env` gains `SegmentBounces func(row string) (count, budget int, inSegment bool, err error)`, a run-wide seam answering a row's segment bounce count and budget from the run's persisted history.
`Env.Slug`'s doc comment adds the Bouncer entry to its readers.
`bouncerEntry` passes `env.Slug` into `BouncerConfig.Slug`, and, when `env.SegmentBounces` is non-nil, a `Bounces` closure that calls it with the row's own name and reports `ok` only for a row in a segment.
Both stay optional: a caller filling neither keeps a working Bouncer whose Reason omits the slug and the budget sentence.

`loomcli/wiring.go` fills `Env.Slug` with `seedSlug(location.WorktreeName)`, and `Env.SegmentBounces` with a closure that reads the status file under its lock (`state.ReadJSONStrict` over the same status path and lock `wire` already holds) and returns `Routing.Bounces(row, st.History)` over `loomrecipe.Routing()`.
The budget is read at call time, never at wire time, because the history grows during the run; an absent status file reports not-in-segment.

Tests:
`entries_bouncer_test.go` — a Bouncer row built with `Env.Slug` and a fake `SegmentBounces` that records the row name it is asked about yields a producer whose circling Awaiting Reason carries the slug and the budget sentence when the fake reports the budget spent;
a row built with neither still constructs.
`wiring_test.go` — the wired `Env` carries the worktree's slug, and its `SegmentBounces` reports the count and budget of a status fixture whose history holds Bouncer Stucks.

**ImpactSummary:** Adds one optional `Env` seam and fills `Env.Slug` for loom; batten entries already read `Slug` and are unaffected.

**Commit:** 9: loom: tell the Bouncer its slug and live bounce budget
