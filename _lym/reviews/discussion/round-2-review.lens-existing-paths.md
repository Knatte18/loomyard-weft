```yaml
lens: existing-paths
outcome: reported
```

existing-paths lens: tracing each state, file and key the record introduces or changes through the code that already writes or reads it.

**Paths traced, per decision**

- D1 and D2, `State.PID` written by `recordStart` (`internal/dobbyengine/loop.go:201`): survives. Start ticks are added beside it.
- D2, `State.PID` also written by `tick` when the record has no pid (`internal/dobbyengine/loop.go:275-278`): **breaks**. That path writes pid and start time with no start ticks, and the record is silent about it.
- D2, readers of `State.PID`: `battencli` arm (`internal/battencli/arm.go:647`) and `DobbyViewReader.Read` (`internal/battencli/dobbyreads.go:331`) read the record tolerantly. Survives, since a new field changes nothing for them.
- D1, the `runproc` record written by `registerDobby` (`internal/beamcli/dobbydeps.go:59-65`), moved behind the lock seam: survives. Teardown and `EndRole` key on owner and role, not on when the record was written.
- D4, the tidy in flight under `UpdateState` (owned exit at `internal/dobbyengine/owned.go:19,48`, land notice at `internal/dobbyengine/landnotice.go:43,58`, tidy at `internal/dobbyengine/tidy.go:109,179`): survives. Each writes only its own fields.
- D5, `geom.LogPath`: written by `SpawnDetached` (`internal/dobbyengine/start.go:258-283`) and the `Deps.Log` seam (`internal/beamcli/dobbydeps.go` `Log:`). No other Go reader exists. Survives.
- D5, the doc text naming stdout as `dobby.log` (`internal/beamcli/doc.go:117`, `docs/overview.md:448`): a doc update covered by the record's docs scope. No finding.
- D8, `countLandings` (`internal/dobbyengine/tidy.go:101-131`) with `CountedLandings` and `LandedSinceTidy` under a machine-filtered `Landings`. New landings survive. **Breaks** for state already on disk: landings from other machines, counted before this change, stay in `LandedSinceTidy`.
- D8, the baseline (`LandingsBaselined`) under the filtered set: survives. A slug filtered out is never marked seen, and that is harmless.
- D9, the `adopt` key against `gateslot.LoadConfig`, which loads strictly against the template (`internal/gateslot/config.go:148`), and against `MigrateConfig` run through `configreg`'s gate row (`internal/configreg/configreg.go:128`): survives.
- D9, the help example `lym gate test --wait --gate finalize` (`internal/gatecli/cli.go:78`) and the adopt docs (`internal/adoptcli/doc.go:73,77`, `docs/overview.md:438`): text updates. Enumerating them is the plan's job, so no finding.
- D10, `suite.log`: `startSuite` truncates it today (`internal/adoptcli/deps.go:207-218`), and the record moves that to the check holding the lock. No other writer or reader exists. Survives.
- D10 and D11, `check.json`: new file with no existing writer or reader. Survives.

**Findings**

1. LOW, design, Decision 2: a record re-filled by `tick` has this pid and no start ticks, so it reads as a re-exec, and the pid-reuse guard does not hold for it.
   - `tick` rewrites a record found without a pid with `os.Getpid()` and start time only (`internal/dobbyengine/loop.go:275-278`).
   - Decision 2 has `recordStart` alone write start ticks, and reads "this pid, no start ticks" as a re-exec, on the premise that only a binary from before this change writes such a record.
   - After a reboot that reuses the pid, a fresh dobby then takes the ten-second wait. If it is refused, it posts the re-exec notice, which is the case the start ticks were added to rule out.
   - The record says nothing about the `tick` path writing a pid.
   - Evidence: `internal/dobbyengine/loop.go:269-278`, and Decision 2 in `_lym/discussion/decision-record.md`.
2. LOW, design, Decision 8: other machines' landings already counted into `LandedSinceTidy` stay there after the fix ships, so the next tidy on that machine still covers them.
   - `countLandings` appends every unseen landing to `LandedSinceTidy` (`internal/dobbyengine/tidy.go:122-128`).
   - Decision 8's bound says only `settleTidy` clears it, and no start resets it.
   - A dobby like Neve's, which counted another machine's landings before this change, keeps them queued, so its first tidy after the deploy still covers them. That is the double tidy part 3 is meant to end.
   - The record is silent on this already-recorded state. The fix is to drop the inherited entries the machine filter excludes, or to state the bound.
   - Evidence: `internal/dobbyengine/tidy.go:101-131`, and Decision 8 in `_lym/discussion/decision-record.md`.

**Tool calls**
1. A grep for `LogPath`, state writes and `suite.log` users, to find the writers and readers of the files the decisions change. The manifest had no caller list for them.
2. `DobbyGeometry`'s tail plus a grep for state, tidy-count and log users, to find every writer of the dobby state record and the tidy count.
3. `countLandings`, the owned-elsewhere exit and the two `battencli` state readers, which sat one hop past the manifest, to trace D4 and D8 and the state readers.
4. A grep of the `dobbyengine` pid and start-time writes and of text that names `dobby.log` or `suite.log`, to find a second pid writer and the text consumers.
5. `loop.go:262-285` and the pid-clearing paths, to read the `tick` pid write the manifest lacked.
6. The `gatecli` help example and the doc lines that name `finalize`, to check the text paths Decision 9 changes.
