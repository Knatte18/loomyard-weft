# Card 5 — `lyx loom start` vouches for the resume it spawns

**Edit:**
- `internal/loomcli/start.go`
- `internal/loomcli/entryobservation.go`
- `internal/loomengine/config.go`
- `internal/loomcli/start_driver_test.go`
- `internal/loomcli/start_midmerge_test.go`

**Intent:**
A resume through `lyx loom start`, including a migration between machines, is no anomaly (decision D4).
In `runDriverSpawnAndWait`, on the `mustSpawn` path only, after the unfinished-merge refusal and the stale park-marker removal and immediately before either arm spawns, start reads the status file (`c.shedPaths.StatusPath`) and writes the handoff voucher with its history length and state through `recordHandoffVoucher`, replacing any earlier voucher.
It writes only when Tier 2 is on (`c.frictionDir != ""`), the gate under which the entry observation consumes it; a missing or unreadable status file, or a failed write, is warned and changes nothing else.
The parked-driver resume, the live-strand no-op and every refusal write no voucher.

What this lets through and what bounds it: the voucher suppresses at most one entry observation, the first one, and only when its history length and state equal what start recorded; it lives under `.lyx` through `loomengine.LoomHandoffVoucher`, so it never crosses machines or survives a fabric re-wire.
A voucher whose spawn then fails its readiness check stays until the next start or completed step overwrites it, and suppresses at most one later matching observation; a driver start spawned that then dies mid-run is not noted, and its evidence stays in the driver log and trace.
State this bound in the doc comment of `recordHandoffVoucher` and of `LoomHandoffVoucher` (`internal/loomengine/config.go`), which now name both writers, and in `runDriverSpawnAndWait`'s doc comment.

Tests, untagged, through the existing fakes:

- `internal/loomcli/start_driver_test.go`: on the llm arm's success path with a seeded running status, Tier 2 on, the voucher holds the status's history length and state, and a voucher already on disk is replaced; with Tier 2 off no voucher is written; a parked-driver resume writes none, and `TestRunDriverSpawnAndWait_LiveDriverOverRunningRunIsANoOp` also asserts the live-strand no-op writes none.
  The one-shot consume and the history-length and state match are already pinned by `TestConsumeHandoffVoucher` and `TestObserveEntry_ConsumesHandoffVoucherIntoObservation` (renamed by card 3), so this card adds no consume-side test.
- `internal/loomcli/start_midmerge_test.go`: the unfinished-merge refusal leaves no voucher.

**ImpactSummary:** `lyx loom start` writes one machine-local file before a spawn; refusals, resumes and exit codes are unchanged.

**Commit:** 5: loom start: vouch for the resume it spawns
