```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checked every bound, behaviour and code fact the fix added or rewrote in D5's Bound, the new D2 criterion and support-log Q5 against HEAD.

No findings.

Claims checked:
- **The old pawl driver calls `orchcli.BeamUp` in-process at its Beam-Up row.** Holds: `internal/loomcli/pawlwiring.go:106-107`, and `BeamUp` reaches `ensureRunsStrand` at `internal/orchcli/start.go:483`.
- **The old `ReplaceStrand` drops the runs strand before the old add refuses.** Holds: `removeStrandLocked` and `killStrandPanes` run at `internal/reedengine/replace.go:65-71`, before `addStrandLocked` refuses a held overview at `internal/reedengine/strand.go:404-409`. The text says "can drop", which correctly allows for a session whose status strand is not yet in the overview, where the replace succeeds.
- **With no runs strand, the old driver replaces the status strand.** Holds: the `loomStatus != nil` branch at `internal/orchcli/runs.go:88-91`.
- **A replaced status strand comes back at the next loom start.** Holds: `start` runs `ensureStatusStrand` on every entry, at `internal/loomcli/start.go:553`, and it adds the strand when it is absent (`internal/loomcli/sharedbootstrap.go:311-315`).
- **The healing loom start replaces an old-key status strand.** Holds: an old build in the sidecar takes the replace path (`resolveStatusStrandAction`, `internal/loomcli/sharedbootstrap.go:292-310`).
- The D2 criterion and the Q5 rewording state design and record facts, not code facts. Neither contradicts the code.

Tool calls beyond my context:
- grep for `ensureRunsStrand`, `BeamUp` and `ensureStatusStrand`, to find call sites my context did not show.
- Read `internal/loomcli/start.go:520-569`, to confirm every start runs `ensureStatusStrand`.
- Read `internal/loomcli/sharedbootstrap.go:270-319`, to confirm the status strand's keep, replace and add behaviour.
