```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing the code paths that already run and touch the machine name, the owners, the machine file or the per-user config dir, to see whether each decision's result survives them.

**Findings**

1. **MEDIUM. D6, race bound.** The record says a check that falls in the window only owner-pauses a pushing run, and that `lym tender resume <slug>` fixes it. The window reaches more paths than that, and their messages name `lym adopt`, which the record rejects for a rename.
   - The orch watcher's `beamOwnerElsewhere` (`internal/orchcli/watch.go`) uses `machinename.Name()` with `boardengine.OwnedElsewhere`, so it judges the orch's own worktree as owned elsewhere.
   - `dobbyengine.OwnedWorktrees` and `EnsureLive` (`internal/dobbyengine/start.go`) compare `primeOwner == machine` and `beam.Owner == machine`. A dobby started in the window skips the prime and beams, or refuses them naming `lym adopt root` or `lym adopt <slug>`.
   - The hibernate wake's `holdRuns` (`internal/hibernateengine/wake.go`) pauses an owned-elsewhere run with `OwnerCaller` and then ends its processes through `EndOwned`. That is worse than a pause.
   - The dobby already has its own owner-pause lift (`beamcli/dobbydeps.go` `LiftOwnerPause`/`liftOwnerPause`). The record doesn't say whether that lift clears the pause after the sweep, so the hand `lym tender resume` it prescribes may be redundant.
   - Fix: name these paths in the bound. State which recover by themselves (the dobby lift) and which do not (ended processes, a dobby or orch refused with an adopt way forward). Or narrow the window, for example by sweeping before the file write.

2. **MEDIUM. D3 and the Slack constraint.** The record claims that new channels take the new name and that long-lived processes pick up the name at their next check. That doesn't hold for the Slack relay. `hubgeom/slackgeom.go:50` calls `machinename.Name()` once while it builds the Slack geometry. The relay keeps `geom.Machine` for its whole life: in `keeper.create` and `retireEntry`, and in `outboundRoute`'s `FormatMachineAddress` sender name (`slackengine/outbound.go:215,222,390`). A running relay therefore creates channels and signs posts under the old name until it restarts. Fix: state the restart, or have the verb name it as a way forward.

3. **LOW. D8.** Pointing `XDG_CONFIG_HOME` at the run directory in `tmuxkit.setTempEnv` also moves `githubclient`'s token cache and the Slack machine files (`hubgeom.SlackMachine`) for every package under `tmuxkit.Main`. The record doesn't mention this. A test tier that reads a real cached GitHub token or Slack file would lose it. The record should say no test relies on those files, or exempt them.

**Paths traced, per decision**

- **D1, D2 (file and directory resolver):**
  - `githubclient.CredentialDir`, used by `hubgeom.SlackMachine` for the Slack files, keeps the same directory if it delegates: survives.
  - `standalonestate.UserStateDir` is untouched: survives.
- **D3 (resolution on every call):**
  - Owner checks in battencli `owner.go`, loomcli `ownercheck.go`, orchcli `watch.go` and `ownerpause.go`, and `dobbyengine/start.go` read it per call: survive.
  - The Slack geometry reads it once per relay: breaks the claim (finding 2).
  - `battencli/arm.go:444` stamps `shed.Machine` once per pass, and D7 covers that.
- **D4 (name rule):**
  - `slackengine.ChannelName` sanitises anyway; `agentname.FormatMachineAddress` is display only: survives.
- **D5, D6 (verb and sweep):**
  - `SetOwners` re-read and `Lost` (`ownerset.go:152-217`): survives with the replace set.
  - The prime-claim writers (`hubgeom/dobbygeom.go:109`, `orchcli/ownerpause.go:138`, battencli `owner.go:200`, beamcli `create.go:163`) write only when no prime owner is recorded, and they write the current name: survive.
  - `pairteardown` `OwnerReader` and fabriccli `remove.go` compare by equality, so they survive after the sweep and break in the window (finding 1).
  - The adopt runreport and holds follow the board: survive after the sweep.
  - Dobby `OwnedWorktrees` and `EnsureLive`, the orch watcher and the hibernate wake break in the window (finding 1).
- **D7 (history match):**
  - Shed history `Machine` is written by `shedengine/run.go:388`, `reroute.go` and `entry.go`. `DobbyLandings` is the only production reader of it: survives.
- **D8 (test isolation):**
  - `setTempEnv` also redirects the GitHub and Slack credential files (finding 3).

**Tool calls**

1. grep of `.Machine` and `Machine:` reads and writes across `internal` and `cmd`: needed to find every reader of the stored name, which my context lacked.
2. `dobbyengine/start.go` `OwnedWorktrees`/`EnsureLive`, orchcli `ownerpause.go`/`watch.go` greps, `outboundRoute`, and the hubgeom Slack and dobby geometry: needed to see how each reads the name, which my context lacked.
3. orchcli `ownerpause.go` lines 25-118 and a grep for `OwnerCaller`: needed for the owner-pause install path and its lifters.
4. `pauserecord/doc.go`, beamcli `dobbydeps.go` `liftOwnerPause`, hibernate `wake.go` `holdRuns`: needed for how owner pauses are lifted and what a wake does to an owned-elsewhere run.
5. The callers of `liftOwnerPause` and orchcli `watch.go` `beamOwnerElsewhere`: needed for the dobby lift's wiring and the orch watcher's owner read.
