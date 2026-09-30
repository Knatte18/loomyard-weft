# Card 27 — loom-gate-refusal-sweep

**Edit:**
- `internal/loomengine/seed.go`
- `internal/loomshed/webster.go`
- `internal/loomshed/batchifier.go`
- `internal/loomshed/discussionwrite.go`
- `internal/loomshed/planwrite.go`
- `internal/preflightshed/preflight.go`
- `internal/shedadapters/webster.go`
- `internal/loomengine/seed_test.go`
- `internal/loomshed/gates_test.go`
- `internal/preflightshed/preflight_test.go`
- `internal/shedadapters/webster_test.go`

**Intent:**
Give every stop of loom's gate rows a way forward (D1) and a reaching test, in the trailing `way forward: …` form.
Loom-Preflight's half-finished stop (`CheckHalfFinished`) is card 22's, and loom's own verbs are card 26's.

Messages to fix:

- `loomengine.CheckSeed`, `seed-incoherent` from an out-of-vocabulary history outcome (a broken adapter's outcome, which Shed records verbatim): seed a new run.
  The clause is attached per failed check, so it never lands on a `CheckHalfFinished` failure.
- Preflight's Stuck reason, per failed check:
  `worktree-clean` → commit or stash the warp's changes with git, and commit the weft's `_lyx` changes with `lyx fabric commit`, then re-step;
  `fabric-sync` (the weft worktree is off its paired branch) → `lyx fabric checkout`, which re-checks out the current warp branch and re-syncs the weft side, then re-step;
  `fabric-ready` and `junction` → `lyx fabric reconcile`, which recreates a missing weft worktree and re-points broken junctions, then re-step.
- The batchifier Stuck reason (Batchifier and Webster rows): fix `batcher.yaml`'s `active:` key, then re-step.
- The commit-seam failures in Discussion-Write, Plan-Write and the Webster row: transient; re-step the row.
- `shedadapters` Webster producer, "paused out of band": re-step the Webster row; `lyx webster run` clears the pause and resumes.

Grouped:

- `CheckSeed` verdicts `seed.go` documents as pre-empted by Shed's step-1 read gate — `seed-missing`, and `seed-incoherent` from a decode failure, an invalid state or a done state — are wiring rows: unreachable through the Loom-Preflight row.
  `seed-missing` and the decode and invalid-state branches are covered by card 25's step-1 refusal rows;
  the done state gets no row anywhere, because step 1 short-circuits a done run without calling a producer, which is not a refusal.
- `seed-unreadable` (a stat failure) is raw I/O.
- The remaining `seed-incoherent` branches (an empty mandatory field, a `current_producer` other than the row, a non-RFC3339 `at`) are unreachable from a status file lyx writes and group as wiring.
- Nil commit seams and closures, the unrecognized webster outcome, context-cancellation errors (a pause, not a refusal), prompt and spec composition errors, preflight's `geometry` failure (unreachable from a worktree lyx created), and a `junction` failure caused by an unreadable `fabric.yaml` (`CauseConfigLoadFailed`, a config decode error).

Tests: each fixed row is reached from a realistic state (a `hubforge` hub for the preflight rows, a status file as Shed writes it for the seed row), the message asserted to contain its way forward, the way forward taken, and the refused row asserted to proceed.
The `fabric-sync` row puts the weft worktree on another branch and takes `lyx fabric checkout`; the `worktree-clean` row dirties the warp and the weft separately.
Tests that spawn git carry the `integration` tag and use `internal/hubforge`.

**ImpactSummary:** Stuck-reason and message text plus tests across loom's gate rows; anomaly and ly-drive classification read state, kind and the budget reason, never these texts.
