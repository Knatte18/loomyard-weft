```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**MEDIUM: D12 says `shuttleengine` needs a new lookup that already exists**
- Changed text: D12, "Shuttle's run record lists each run's output files, so a run is found by its report path even when state never recorded its guid; `shuttleengine` gains that lookup beside `FindRun`."
- Problem: the lookup by output file is already there. `Runner.Attach`, `AttachGated`, `AttachIfLive(Gated)` and `ProbeGated` find a live run whose `run.json` declares the spec's output-file set, with no guid.
- Evidence: `internal/shuttleengine/attach.go:35-44` says Attach "looks for a still-live, never-terminated run matching spec's output files". `attach.go:112` calls `collectAttachCandidates(root, normalized.OutputFiles)`.
- Fix: drop "gains that lookup beside `FindRun`" and name `AttachGated`, or `ProbeGated`, as the attach.
- Related: `Attach` removes superseded respawn-eligible strands before it answers not-found (`attach.go:42`, `:52-57`). D12's "only then does it treat a run that is gone per D6" should allow for that.

**LOW: D11 says every fingerprint helper takes `*State` and the run directory**
- Changed text: D11, "the fingerprint helpers take `*State` and the run directory".
- Problem: true of `changedPlanFiles`, `PlanEditError`, `restampFingerprint` and similar. Not true of `fingerprint(planDir)`, `Fingerprint(planDir)`, `planFileHashes` or `overviewFrameHash`, which take only the plan directory.
- Evidence: `internal/websterengine/fingerprint.go:23`, `:66`, `:86`, `:175`.
- Effect: the conclusion that the state types must move still stands, so a reader is only misled about the plain hashers.

**Claims checked and found true:**
- `RunState.OutputFiles` is persisted.
- `tackler_max_spawns` and `arbiter_max_spawns` exist (`internal/sheddriver/config.go:52,58`).
- `lym loom pause --after` exists (`internal/loomcli/cli.go:439`).
- `gatefixer.RoleVerifyFixer` exists.
- Webster's `recovery_timeout_min` default is 60, which matches the new `fixer_timeout_min` default.
- `postBatchInputs`, `checkRecoveredSuspects` and `failureDigestBlock` have the signatures D11 quotes.
- `State.MasterStrand` exists.
- `lym fabric merge-in` calls `websterengine.RunInFlight` (`internal/fabriccli/merge_verbs.go:28`).
- The shed status value `running` exists.

**Outside this lens:** if the row's process crashes, the shed status may still read `running`. `lym weaver reset` would then refuse, and `lym loom pause --after` may not unblock it. That belongs to the fix-way-forward lens.

**Tool calls:**
1. Grep for `RunInFlight` callers, the `merge-in` verb and `OutputFiles` uses in shuttle, to check the merge-in and output-file claims. My context held neither.
2. Read `attach.go`'s function list and the `attach` body, to confirm the output-file lookup already exists. My context had not read `attach.go`.
