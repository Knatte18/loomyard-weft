```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every path that writes, reads or reapplies the state D1–D8 introduce or change.

**Paths traced, per decision**

- D1/D3 machine file, per-call readers.
  - `beamcli/dobbydeps.go:127,191,389`, `dobbyengine/owned.go:91`, `loop.go:555` and `start.go:92,127` read the name on every call: survives.
  - `battencli/owner.go:126` `thisMachine`, `orchcli/watch.go:60`, `fabriccli/anchordeps.go:382`, `fronttask/hub.go:92` and `hubgeom/dobbygeom.go:103` also read on every call: survives.
  - The error text that wraps a D3 failure breaks (F1).
- D1/D3, readers that resolve the name once.
  - The Slack relay through `hubgeom/slackgeom.go` `SlackGeometry`: survives, and D3's bound names it.
  - A `lym batten run` process's `s.Machine` stamps history entries (`shedengine/run.go:388`). A Land written under the old name after a rename survives through D7.
- D1, other files in the same directory: `githubclient` `credentials.json` and the Slack files under `hubgeom.SlackMachine` write only their own names. Survives.
- D1, environment: no production spawn clears `HOME` or sets `XDG_CONFIG_HOME`; the grep found none. Survives.
- D2, `githubclient.CredentialDir` delegating to `UserConfigDir`: its one production caller, `hubgeom/slackgeom.go:21`, keeps its path. Survives.
- D5/D6, `SetOwners` replace set.
  - Other `SetOwners` callers leave the new field unset: `dobbygeom.go:109` ClaimPrime, the orch's `claimPrime`, adopt and beam start. Survives.
  - `writeOwners` skipping done entries (`ownerset.go:188`): survives, as D6 states.
- D6 window, `dobbyengine.OwnedWorktrees` and `EnsureLive` (`start.go:91-135`): the prime is skipped as owned elsewhere until the sweep runs. D6's `lym beam start root` recovery covers it.
- D6 window, the dobby probe's local-Bolt lift (`dobbyengine/doc.go:173-174`) against the resume's origin verdict when the sweep commits but its push fails: breaks (F2).
- D6, prime claim at dobby start and orch bring-up: it writes only when no prime owner is recorded. Survives.
- D6, `lym adopt` mid-rename: adopt records the old name, the sweep rewrites it, and adopt's green check reads the current name on each call. Survives.
- D7, `battencli.DobbyLandings`: its one caller is `beamcli/dobbydeps.go:192` through `machineLandings`, which already reads the name on each call. Survives.
- Persisted machine fields: the only Go-written record field holding the name is `shedengine/status.go:120` `HistoryEntry.Machine` (D7). No yaml or json record keys trust or locks by machine name. Survives.

**Findings**

- **F1**
  - Severity: MEDIUM. Class: design.
  - Location: D3 (error policy), with `PATTERN-refusal-way-forward`.
  - Summary: D3 makes `machinename.Name` fail on a bad machine file or a relative `HOME`/`LOCALAPPDATA`, and its error names the way forward. Existing wrappers that run in every batten pass still prescribe the hostname fix. `battencli.thisMachine` wraps every such error as "this machine's name is unknown … fix the hostname, then re-run lym batten run &lt;slug&gt;". `boardengine.SetOwners`'s empty-machine refusal says the name "fails when the hostname cannot be read". The record says nothing about the way forward surviving those wrappers, so a shed refusal would name the wrong one. D3 should state that a caller wrapping the name error keeps the error's own way forward and names no hostname fix.
  - Evidence: `internal/battencli/owner.go:126-134`, `internal/boardengine/ownerset.go:82`.
- **F2**
  - Severity: LOW. Class: design.
  - Location: D6 (sweep failure, and the window's recovery list).
  - Summary: a hub whose sweep commits but whose push fails keeps the owner commit locally. The dobby's probe reads the local Bolt with no pull and finds the new name, so it lifts its owner pause and runs a full pass. That pass's resume judges against origin inside `ReconcileOrigin`, finds the old name and pauses the run again. The run flaps between pause and full pass until a re-run pushes. D6 reports the failed hub and the re-run, but its recovery bullet ("after the hub's sweep, the dobby's probe … lifts its own owner pause") does not cover a sweep that committed and never pushed.
  - Evidence: `internal/boardengine/ownerset.go:278` (commit kept locally on a failed push), `internal/dobbyengine/doc.go:173-174` (probe reads the local Bolt with no pull), `pattern/PATTERN-owner-machine.md:35` (resume verdict judged against origin).

**Tool calls**

1. grep of how `dobbygeom`, `orchcli` `watch`/`start`, `dobbydeps`, `fronttask` and `anchordeps` call `machinename.Name`, plus the `DobbyLandings` callers: the manifest listed these consumers but not how each takes the name (once or per call).
2. grep of how `dobbyengine` and `battencli` call `Machine()`: needed to rule out a cached name in the long-lived dobby and batten processes.
3. grep of yaml/json machine fields, plus `battencli/owner.go:120-135` and `dobbyengine/start.go:85-135`: needed to rule out other persisted name fields and to read the wrapper text and the start-time owner paths.
4. grep for spawns that clear the environment or set `XDG_CONFIG_HOME`, and for machine use in creel: needed to check whether any lym process resolves a different config directory than D1 assumes.
