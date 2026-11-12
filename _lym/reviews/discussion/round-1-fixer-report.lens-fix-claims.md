```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, that every claim the fix added or rewrote holds against the code at HEAD.

1. MEDIUM. D7's new sentence "Every reader that compares a map's `commit` with HEAD to mean the map is current reads the planned HEAD, the exit record's `planned_head`, instead. The audit's candidate filter (`internal/gatecli/testmapaudit.go`) is one such reader."
   - **Problem:** taken literally, the audit would read a stale value, because the exit record is written only when a pass ends.
   - **Evidence:** `internal/gatecli/testmap.go:695` writes `PlannedHead: r.head` into the exit record at the pass's exit. The audit runs inside the pass, before that write, so when it reads the record it gets the previous pass's `planned_head`, not this pass's HEAD. Inside the pass, the planned HEAD is `r.head`, and every map the pass planned without a miss stands current at it. The filter `m.Commit != r.head` (`testmapaudit.go:38`) has to accept those maps. The sentence names the wrong source for an in-pass reader. Only a reader outside the pass, such as the dobby at `internal/dobbyengine/testmap.go:89`, reads the record.

2. LOW. D7 Bound: "a rewrite of the prime's code branch is reset as a rewrite."
   - **Problem:** the package doc says "rewind", not "rewrite", and the bound leaves out that the dropped commit's object survives.
   - **Evidence:** `internal/fabricengine/doc.go` reads "The prime's code branch is the operator's, so a rewind of it resets as a rewrite does." The same doc says "`git fetch` never prunes objects, so a rebased-away commit's object survives the fetch." The degradation the bound states still holds: `testselect.go:269-271` gives `MissCommitUnreadable`, and `upkeep.go:74-75` gives `UpkeepMiss`.

Claims checked and found to hold:
- D9: `testMapRunning` lists only `runproc.HubOwner` (`dobbydeps.go:619-625`), so the prime's dobby counts only the prime's pass. Teardown ends the processes under its slug through `runproc.EndSparing(..., slug, ...)` (`teardown.go:698`).
- D6: `HubEnvDigest` hashes `hubEnv.Entries()` (`testmappackage.go:818`), loaded through the layered `configengine.LoadOrTemplate`, and `vars` is `CategoryMachine`.
- D4: map locks follow `lock.TrackedFileLockPath`.
- D8: `writeJSONUnlocked` always indents and keeps unknown keys.
- D7: `MissCommitUnreadable` exists.

Further tool calls (one):
- A grep and `sed` read of `testMapRunning` in `internal/beamcli/dobbydeps.go`, the rewrite paragraph of `internal/fabricengine/doc.go`, `stampEnvironment`, and `envconfig.LoadConfig`. My context did not hold their bodies.
