# Card 8 — stopped-young-run-dir-rows

**Edit:**
- `internal/shuttleengine/attach_test.go`

**Intent:** Pin the shape of incident #452 in the attach probe's tables, with no production change (D5): a run dir of a strand lyx itself stopped is respawn-eligible at once.
`TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge` adds `runOutcomeStopped` to the outcome list of both of its sub-tables, the untracked strand and the cleared pane binding, so a `stopped` record younger than the age bound answers not found with no error.
`Attach` and `ProbeGated` run the one probe body, so these rows pin the not-found answer the burler half probe receives as well, and no `ProbeGated` case is added.
The burler half probe sees only shuttle's not-found answer, never the record behind it, and `TestEngine_ProbeRound`'s existing rows, a half with no live run and no output file is gone and one with its output file present is done, are the burler coverage D5 asks for, so no burler row is added.

**ImpactSummary:** Test-only; no production symbol changes.
