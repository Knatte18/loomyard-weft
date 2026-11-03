# beam create clears a done motif run under the same slug

A task that ran recipe `motif` to write its Sketch and was then turned into a project could not start its beam.
`lym beam create <slug>` refused with `"<slug>" has already completed`, and the only way forward was `lym batten cleanup --apply` from the prime.
A motif run is the planned first step of a project split, so its done record should not block the project's own pawl run.

## What changes

- `lym beam create` now removes a done batten run under the slug before it starts the pawl run, when the run's pair is gone and its child ran `motif`.
  The envelope reports it as `motif_run_cleared`.
  Any other done run is still refused, naming `lym batten cleanup --apply`.
- `battencli.ClearMotifRun` does the removal.
  It takes the slug's run lock and status lock, reuses the records removal `lym batten cleanup` uses, and spawns the records push afterwards.
  It leaves alone a run whose lock is held, that is not done, or whose child recipe is not `motif`.
- Seed-Child now writes a child-recipe record (`shedrun.WriteChildRecipe`, read by `shedrun.ReadChildRecipe`) into batten's own run directory.
  The task worktree's seed is gone once the pair is torn down, so this record is how a done run still tells which recipe its child ran.
  The records commit carries it with the status and seed.
- Docs (`docs/overview.md`, the `beamcli`, `battencli` and `shedrun` package docs) and the `beam create` and `batten cleanup` help text name the new step.

## Design decisions

- No backfill.
  A done motif run seeded before this change has no child-recipe record and keeps the old refusal; no legacy path reads the seed or board type to clear it.
  The one such run, `codex-engine`, was already cleared by hand, and `lym batten cleanup --apply` stays the way forward for any stray one.
- The child recipe is a durable record in batten's run directory rather than a lookup in the task worktree, because the clear only runs once the pair is gone.
- A removal that fails before its records commit lands writes the status and child-recipe records back and returns the error, so a re-run finds the run again.
  A failed push spawn or scratch emptying only warns, since the done record is gone either way.

## To check by hand

- `ClearMotifRun` on a failed commit: when the removal fails partway, `restoreMotifRunRecords` writes back the status and child-recipe records, but the half-removed state (for example a commit that failed after some files were removed) was not exercised against a real records repository.
  See `internal/battencli/cleanup.go`.
- The Seed-Child write of the child-recipe record has a pinning test, but its error path (the write failing after the child is seeded) has no test of its own.
  See `internal/battencli/wire.go`.
- Not run by hand: `lym beam create` against a real project whose slug has a done motif run.
