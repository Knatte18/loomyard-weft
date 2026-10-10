# Card 8 — stopped-young-run-dir-rows

**Edit:**
- `internal/shuttleengine/attach_test.go`

**Intent:** Pin the shape of incident #452 in the attach probe's tables, with no production change (D5): a run dir of a strand lyx itself stopped is respawn-eligible at once.
`TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge` adds `runOutcomeStopped` to the outcome list of both of its sub-tables, the untracked strand and the cleared pane binding, so a `stopped` record younger than the age bound answers not found with no error.
`TestProbeGated_ReturnsHandleWithoutWaiting` gains a sub-case `stopped_young_record_is_not_found`: an untracked strand whose record reads `stopped`, with its output file absent and its directory younger than the age bound, answers `(nil, false, nil)`; this is the probe the burler half probe consults, and the burler engine's own `TestEngine_ProbeRound` rows already map a not-found probe to a gone half, or a done one when the output exists, so no burler row is added.

**ImpactSummary:** Test-only; no production symbol changes.
