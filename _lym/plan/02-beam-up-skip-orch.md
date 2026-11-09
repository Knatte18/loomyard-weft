# Card 2 — beam-up-skip-orch

**Edit:**
- `internal/orchcli#BeamUpOptions`
- `internal/orchcli#orchCLI.beamUp`
- `internal/orchcli#TestStart_StartsTheDobbiesFirst`

**Extends:**
- `internal/orchcli/doc.go`

**Intent:**
D7: `orchcli.BeamUpOptions` gains the field `SkipOrch bool`.
Its comment says only `lym adopt` sets it, for a request carrying `--no-orch`.
`orchCLI.beamUp` with `SkipOrch` set runs every step up to the orch as today — the lock, the refusal, the dobby, the relay, reed and the runs strand — and then returns without calling `startOrch`.
The report's orch fields stay empty, and it carries no warning for the skip.

What the field lets through: the beam comes up with its dobby and no orch.
Its bound: only adopt sets it, the dobby always starts, and `lym beam start` starts the orch later.

`TestStart_StartsTheDobbiesFirst` gains a `skipOrch` input column, a `wantNoOrch` expectation column and one tier0 row.
The table today asserts only `wantFirst` and `wantWarning`, so `wantNoOrch` asserts on the calls the test's fake starter records: with it set, no strand launch and no watcher spawn is among them.
With `SkipOrch` set, the dobby seam is called first, and the strand is neither launched nor its watcher spawned.
The existing rows, with the field unset, still launch the strand, which pins the default half of D7.
The orchcli `doc.go` gains one sentence on `SkipOrch`.

**ImpactSummary:** one new opt-in field; the existing callers in beamcli, adoptcli and orchcli leave it false, so no other behaviour changes.
