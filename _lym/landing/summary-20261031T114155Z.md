# Dobby lands a beam once it is flagged ready and nothing works under it

A beam's dobby read "nothing works under the beam" as "the beam is done".
So a newly created beam with no work yet made its dobby ask the orch to run `lym beam land`.
A beam now carries a ready flag, and only a flagged beam is landed.

## What changes

- **`lym beam ready <slug>`** (new) flags a beam ready to be merged into its parent.
  It writes a `pawlengine.ReadyRecord` (`slug`, `set_at`) to `_lym/landing/beam-ready.json` in the beam's pair, beside the change description, and commits the landing directory.
  It runs from anywhere in the hub, as `lym beam land` does, and refuses what `land`'s entry check refuses.
  A beam already flagged is reported as `already`, and nothing is written.
  The beam's orch or the operator runs it; dobby never does.
- **Dobby's landing step** replaces the can-land notice (`internal/dobbyengine/land.go`, renamed from `landnotice.go`).
  Once per episode in which the beam is flagged and nothing works under it, the dobby calls the new `Deps.Land` seam.
  That seam runs the `lym beam land` sequence itself and writes the land request the pawl run consumes at Beam-Await.
  The dobby posts a notice to the beam's orch only when it cannot land the beam: another machine owns the parent (naming `lym adopt <subject>`), or `Land` refused (carrying the refusal and its way forward).
  An unflagged beam reads as not passing however long it has been idle, so it gets neither a landing nor a notice.
  The state record's `land_notice` is replaced by `land` (`State.Land`, `LandState{At}`).
- **`lym beam land`** refuses an unflagged beam after its owner checks and before the work-remaining refusal, naming `lym beam ready <slug>`.
- **Beam-Await's re-check** (`pawlRecheckLines`) adds a line for an unflagged beam, so a land request written by any route cannot land one.

## Design decisions

- The record names its beam.
  A pair forked from a flagged beam inherits the beam's `_lym` content, and `pawlengine.ReadyFor` reads a record naming another slug as no flag.
  `_lym` never merges into the parent, so a landed beam does not mark its parent ready either.
- The record's paths, `loomengine.BeamReadyPath` and `BeamReadyLockPath`, sit with the landing directory that `loomengine` already owns.
  The lock is at the mirrored `.lym/landing/` path.
  The record goes through `internal/state`.
- A refused landing is not retried within the same episode.
  The notice carries the refusal's way forward, and a reading that stops passing re-arms the step.
- No verb clears the flag.
  A beam flagged early still lands only once nothing works under it, because the work-remaining refusal and Beam-Await's re-check still stand.
- `onHubLandDeps` in `internal/beamcli/cli.go` (renamed from `landFromHub`) resolves the hub for both `land` and `ready`.
  `judgeLandEntry` now takes the verb name that opens its refusals.

## Tests

- `TestReadyFor`: an absent record, the beam's own record and another beam's record.
- `TestLand` gains an unflagged-beam row; `TestMarkReady` covers the entry refusals, the first write and an already-flagged beam.
- `TestPawlRecheckLines` gains an unflagged-beam row.
- `TestLandBeam` covers landing once per episode, no step for a reading that does not pass, the parent-owned-elsewhere and refused-landing notices, re-arming, a restart, the prime's dobby, a failed post and a failed read.
