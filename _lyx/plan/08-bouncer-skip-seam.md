# Card 8 — bouncer-skip-seam

**Edit:**
- `internal/shedadapters/bouncer.go`

**Create:**
- `internal/shedadapters/bouncer_skip_test.go`

**Intent:**
Give the generic Bouncer an optional skip seam (D7), so a review segment can settle as approved without spawning a judge when its caller says the artifact needs no review.

`BouncerConfig` gains `Skip func() (bool, error)`; nil is the absent value and keeps every existing row behaving exactly as before.
`Bouncer.Call` consults it first, after the entry check and before resolving the round:

- true: settle as approved without a seed or judge spawn, calling `Approve` then `Commit` (each when non-nil) with the same failure disposition `settle`'s approved branch gives them, logging an `Info` line naming the skip, and returning Done with an empty pointer;
- false: review exactly as today;
- an error: log a `Warn` naming the producer and the cause, and review exactly as today, because the seam is an optimisation and a guard that does not protect correctness warns rather than halts.

Bound: the seam can only approve what its caller already classified; it cannot reject, re-route, or touch the run directory, and a nil or failing seam always falls back to a real review.

Tests in the new file, with a recording fake shuttle: a true seam spawns nothing, calls `Approve` before `Commit`, and returns Done; a false seam seeds round 1 exactly as an unconfigured Bouncer does; an erroring seam seeds round 1 too; a failing `Approve` under a true seam skips `Commit` and fails the way `settle` does.

**ImpactSummary:** one optional config field and one early branch in `Bouncer.Call`; every Bouncer row that leaves the field nil is unchanged.
