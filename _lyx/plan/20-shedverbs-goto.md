# Card 20 — shedverbs-goto

**Edit:**
- `internal/shedverbs/spec.go`
- `internal/shedverbs/verbs.go`

**Create:**
- `internal/shedverbs/goto.go`
- `internal/shedverbs/goto_test.go`

**Uses:**
- `internal/shedengine/goto.go`

**Intent:**
Add `goto` as the fifth generic verb body, so the shed, loom and batten subtrees all gain it from one implementation (D11, Shed Verb-Set Invariant).

`spec.go`: `VerbTexts` gains `Goto VerbText`.
`verbs.go`: `Verbs` returns `run`, `step`, `status`, `pause`, `goto`, in that order; its doc comment names five verbs and the `--to` flag.

`goto.go` builds `gotoCmd(texts, spec)`:
`clihelp.ShouldAbort` first;
a `--to <producer>` string flag, whose absence refuses through `output.Err` naming the valid producers from `spec.Routing.Producers`;
`ensureStatusLockDir` when `spec.EnsureStatusLockDir`;
then `shedengine.Goto` with `spec.StatusPath`, `spec.LockPath`, `spec.StatusLockPath`, `spec.Routing.Producers` and the flag.
Success is `output.Ok` with `status_file`, `run_id`, `current_producer` and `state`.
Every refusal is one `output.Err` line carrying the engine's message.
It never calls `spec.BuildShed`, so a nil one is legal, as for `status` and `pause`.
It adds no step refusal kind and touches no step envelope key.

`goto_test.go` (untagged, reusing `testsupport_test.go`'s seeded status helpers) covers: `goto --to <row>` over a blocked status → exit 0, the envelope's `current_producer` and `state: paused`, the file updated; no `--to` → exit 1 listing the valid names; an unknown row → exit 1 listing the valid names; the run lock held → exit 1 naming `lyx shed pause`.

**ImpactSummary:** Verbs grows from four to five commands; every caller indexes the first four by position, so the three subtrees compile unchanged until card 21 registers the fifth.
