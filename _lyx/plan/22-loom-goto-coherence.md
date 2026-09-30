# Card 22 — loom-goto-coherence

**Edit:**
- `internal/loomengine/coherence.go`
- `internal/loomengine/coherence_test.go`
- `internal/loomshed/loompreflight.go`
- `internal/loomshed/loompreflight_test.go`

**Create:**
- `internal/loomcli/gotoentry_test.go`

**Uses:**
- `internal/shedengine/goto.go`

**Intent:**
Make loom a correct reader of the `goto` history outcome, and give Loom-Preflight's half-finished refusal a way forward (D11).

- `coherence.go`: the history-outcome rule accepts `shedengine.Goto` beside `Done`, `Stuck` and `Awaiting`.
- `loompreflight.go`: when the report carries a `CheckHalfFinished` failure, the Stuck reason ends `way forward: "lyx loom goto --to Discussion-Write" resumes the run past Loom-Preflight, or seed a new run`.
  The row name comes from `NameDiscussionWrite`, the row `Loom-Preflight`'s `on_done` names in the recipe; a comment says so.
  Other failure kinds keep today's reason.

Tests:

- `coherence_test.go`: a history carrying a `goto` entry raises no invalid-outcome failure.
- `loompreflight_test.go`: a status file written by `shedengine.Goto` to `Loom-Preflight` over a history that has passed it (for example a `Discussion-Write` done entry) → Stuck, and the reason contains `lyx loom goto --to Discussion-Write`.
  After `shedengine.Goto` to `Discussion-Write`, `Discussion-Write` is the current producer and the Loom-Preflight row is not re-entered.
- `gotoentry_test.go` (untagged, `package loomcli`): a status file with a live history, moved by `shedengine.Goto`, observed through `observeEntry` with the run lock free, yields no anomaly from `loomengine.DetectCrashResume`, because `goto` writes `paused`, not `running`.

**ImpactSummary:** Loom's seed-coherence check widens its outcome vocabulary by one value, and one Stuck reason gains a suffix; no other loom reader validates history outcomes.
