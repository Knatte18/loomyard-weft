# Card 26 — loom-refusal-sweep

**Edit:**
- `internal/loomcli/approve.go`
- `internal/loomcli/start.go`
- `internal/loomengine/seed.go`
- `internal/loomshed/webster.go`
- `internal/loomshed/batchifier.go`
- `internal/loomshed/discussionwrite.go`
- `internal/loomshed/planwrite.go`
- `internal/preflightshed/preflight.go`
- `internal/shedadapters/webster.go`
- `internal/loomcli/approve_test.go`
- `internal/loomengine/seed_test.go`
- `internal/loomshed/gates_test.go`
- `internal/preflightshed/preflight_test.go`
- `internal/shedadapters/webster_test.go`

**Intent:**
Give every loom refusal and gate stop a way forward (D1) and a reaching test, in the trailing `way forward: …` form.
Loom-Preflight's half-finished stop is card 22's.

Messages to fix:

- `approve`, run not awaiting or blocked at Publish: `lyx loom status` shows where the run is; approve once it halts at Publish.
- `approve`, no pull request, or pull request not open: step the run to Publish (`lyx loom step`), which opens it.
- `start`, "driver did not take the run lock": re-run `lyx loom start`; the named driver log says why the driver exited.
- `loomengine.CheckSeed` reasons: `seed-missing` → `lyx loom start`; `seed-unreadable` and `seed-incoherent` → seed a new run.
- Preflight's Stuck reason, per failed check: `worktree-clean` → commit or stash the warp's changes with git, then re-step;
  `fabric-ready`, `fabric-sync` and `junction` → `lyx fabric reconcile`, then re-step.
- The batchifier Stuck reason (Batchifier and Webster rows): fix `batcher.yaml`'s `active:` key, then re-step.
- The commit-seam failures in Discussion-Write, Plan-Write and the Webster row: transient; re-step the row.
- `shedadapters` Webster producer, "paused out of band": re-step the Webster row; `lyx webster run` clears the pause and resumes.

Already carrying a way forward, test-only where untested: approve's missing status file and its HEAD-differs refusal, start's not-parked refusal, loom's step busy refusal, and the missing-seed refusal.

Grouped: nil commit seams and closures, the unrecognized webster outcome, context-cancellation errors (a pause, not a refusal), prompt and spec composition errors, and preflight's `geometry` failure (unreachable from a worktree lyx created).

Tests: each fixed row is reached from a realistic state (a hubforge hub for the preflight and approve rows, a status file as Shed writes it for the seed rows), the message asserted to contain its way forward, the way forward taken, and the refused row or verb asserted to proceed.
Tests that spawn git carry the `integration` tag and use `internal/hubforge`.

**ImpactSummary:** Message and Stuck-reason text plus tests across loom's CLI, engine and gate rows; anomaly and ly-drive classification read state and kind, never these texts.
