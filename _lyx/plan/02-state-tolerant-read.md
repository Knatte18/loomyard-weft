# Card 2 — Read run records with unknown fields accepted

**Edit:**
- `internal/state#ReadJSONStrict`
- `internal/state#ErrDecode`
- `internal/state#TestReadJSONStrict`
- `internal/loomshed#Seed`
- `internal/loomengine#VerifySeedOwnership`
- `internal/loomcli#poisonStatusFile`
- `internal/loomcli#TestSmokeRunStandalone`
- `internal/loomcli#TestSmokeBootstrapLifecycle`
- `internal/shedengine#TestRun_UnknownTopLevelKeyBeforeRead`

**Intent:**
D1: `state.ReadJSONStrict` stops calling `DisallowUnknownFields`, so the read gate of every standalone shed `status.json` reader in batten, loom and shed accepts a key this binary does not know.
It decodes with plain `json.Unmarshal`, as `readJSONUnlocked` does; a malformed file is still `ErrDecode`, and it still creates no missing parent directory.
Its doc comment drops "rejecting unknown fields" and states what still sets it apart from `ReadJSON`: no `os.MkdirAll`.
The name stays until card 4 renames it, so this card changes behavior only.
`ErrDecode`'s doc comment becomes "a decode failure (malformed JSON)".

Tests:
- `TestReadJSONStrict`'s `UnknownField` row flips to `wantOK: true, wantName: "widget"` with no error, so the row fails if the strict decode returns.
- `TestRun_UnknownTopLevelKeyBeforeRead` pins the refusal D1 removes, so it flips in place: the seeded status with `unexpected_key` now runs, `Run` returns `RunDone` with no error and the producer is called once.
  Its doc comment says the read gate accepts a key that was on disk when it ran; card 3 adds what the persist does with that key.

Comments the change makes wrong:
- `loomshed.Seed`'s comment on the decode-failure branch names "an unknown top-level field" as undecodable and says the driver's read gate rejects that shape; it now names malformed JSON alone.
- `loomengine.VerifySeedOwnership`'s doc lists "malformed JSON or an unknown field" as a decode failure; it now names malformed JSON alone.

The tmux-tier smoke rig `poisonStatusFile` relies on the strict read rejecting an unknown key, so it would no longer fail the driver's read gate.
It instead rewrites the seeded status file's `state` to an unrecognised value (`"poisoned"`), the shape `internal/shedengine#TestRun_UnrecognisedState` pins `Shed.Run` refusing before any producer runs or any persist happens.
`loomshed.Seed`'s `UpdateJSON` still decodes that file and takes its found branch, so the bootstrap tolerates it as before.
Its doc comment and the rig comments inside `TestSmokeRunStandalone` and `TestSmokeBootstrapLifecycle`, which describe the unknown-field shape and the strict decoder, describe the unrecognised state instead.
That tier is compiled here by `go vet -tags tmux` and run by Publish.

**ImpactSummary:** every caller of `ReadJSONStrict` across batten, loom, shed and pairteardown now accepts unknown keys; one shedengine test flips, and the tmux smoke rig changes its poison shape.

**Commit:** 2: read run records with unknown fields accepted
