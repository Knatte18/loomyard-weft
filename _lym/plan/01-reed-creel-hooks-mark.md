# Card 1 — Reed carries the opaque CreelHooks mark

**Edit:**
- `internal/reedengine#Strand`
- `internal/reedengine#AddSpec`
- `internal/reedengine#Engine.addStrandLocked`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestSaveState_ThenLoadState_RoundTrips`
- `internal/reedengine#TestStrandOps_RealTmux`

**Extends:**
- `internal/reedengine/doc.go`

**Intent:**
D5 needs a mark on every strand whose session runs creel's Stop delivery, kept through reed's verbatim resume, so creel can tell a strand the bell may wake from a hand-added one.
Add the opaque field `CreelHooks bool` (JSON `creelHooks`, omitempty) to `Strand`, beside `Provider` and documented like it: its spawner sets it, reed stores it and never reads what it means, and a strand recorded before the field existed, or added outside shuttle, decodes with none.
Add the same field to `AddSpec`, "stamped verbatim onto the new strand's CreelHooks", and copy it in `addStrandLocked` beside `Provider`.
`ReplaceStrand` goes through `addStrandLocked`, so a replacement takes the spec's value with no change of its own; a resume replays the persisted record, so the mark survives it.
`TestAddStrandLocked` sets `CreelHooks: true` on its spec and asserts every added strand carries it and that it survives the SaveState/LoadState round trip the test already runs.
`TestSaveState_ThenLoadState_RoundTrips` sets `CreelHooks: true` on its first strand so the field's JSON key round-trips; its second strand keeps none, pinning the omitted default.
`TestStrandOps_RealTmux`, whose relaunch row already pins `Provider` kept on the relaunched strand, adds `CreelHooks: true` to that strand's spec and asserts the relaunched strand still carries it, so the mark's survival of a resume is tested rather than argued.
The reed package doc gains one sentence beside the provider tag's: the strand carries the opaque `CreelHooks` mark its spawner sets.

**ImpactSummary:** two struct fields and one copy inside reedengine; no caller changes, since every other AddSpec builder leaves the new field at its false zero value.
