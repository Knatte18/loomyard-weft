```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every path that writes or reads the state the Decisions change, and checked whether each Decision survives it.

**Paths traced, per decision**

- **D1/D2, the config directory:**
  - `githubclient.cacheDir`/`credentialFilePath` keep the token cache path if `CredentialDir` delegates: survives.
  - `hubgeom.SlackMachine` reads `CredentialDir` for the Slack files: survives.
- **D3, the per-call read of `machinename.Name`:**
  - Every owner-check caller resolves the name at each check: dobby `Deps.Machine` (`beamcli/dobbydeps.go:127`, `dobbyengine/start.go:92,127`, `loop.go:555`, `owned.go:91`), orch watch and owner pause, battencli dobbyreads and cancel, tendercli `Owners`, fronttask, loomcli landing and pawl, fabriccli anchordeps. Survives: they pick up the new name at their next check.
  - `hubgeom.SlackGeometry` resolves the name once and the relay keeps it (`slackengine/outbound.go:215,222,390`, `keeper.go:132,423`). Survives as D3's bound and the Slack constraint state it.
- **D5, the file write:** the only writer is the new verb, and no existing path writes `machine.json`. Survives.
- **D6, owners rewritten by the sweep:**
  - `boardengine.writeOwners` with `Takeover` is used only by `lym adopt`: survives.
  - Dobby's `ClaimPrime` on start (`dobbyengine/start.go:105,158`) during the window reads the old prime owner as elsewhere. D6 lists `lym beam start root` for it: survives.
  - `PauseOwnedElsewhere`, reached from `loomcli/prepush.go:79`, `battencli/wire.go:1248` and the orch root pre-run, pauses the run **and ends its processes**. **Breaks**: see F1.
  - The tender lift's pair-push skip for pairs owned elsewhere (`tenderengine/doc.go:76`) during the window. **Breaks**: see F2.
  - The hibernate wake's `EndOwnedElsewhere` (`hibernatecli/deps.go:133`) is listed in D6: survives.
- **D7, `HistoryEntry.Machine`:**
  - `battencli.DobbyLandings` is the only non-test reader comparing it (grep over `.Machine ==` found no other).
  - Its one production caller, `beamcli/dobbydeps.go:192`, passes `machine`, so it must also get the former names. Survives under D7.
- **Cross-machine address:** `agentname.FormatMachineAddress` is used only by the Slack outbound, under the relay's once-filled geometry. Survives as D3's bound states.

**Findings**

1. **F1. MEDIUM, design. D6's window bound, first recovery bullet.**
   - D6 says a run that takes an owner pause during the window recovers through the dobby's owner-pause lift or `lym tender resume &lt;slug&gt;`.
   - But `tendercli.PauseOwnedElsewhere` also ends the run's registered processes once the pause stands. A resume only sends a carry-on line to sessions that are still live and restarts nothing.
   - So D6's bound misstates what the window does to such a run. The recovery it names does not bring back the ended agents and driver.
   - Evidence: `internal/tendercli/deps.go:230,241` ("Once the pause stands, it ends the run's owned processes"); callers at `internal/loomcli/prepush.go:79` and `internal/battencli/wire.go:1248`; `internal/tenderengine/doc.go:56-58` (resume sends lines only to live strands).
   - Fix: D6 states that the owner pause ends the run's processes, and names the step that restarts them, with evidence of what restarts the run after the lift.

2. **F2. LOW, design. D6's advised hub pause, and the recoveries that use `lym tender resume`.**
   - A lift skips the deferred pair push of every pair `Deps.Owners` reads as owned by another machine.
   - So a `lym tender resume` run before that hub's sweep has stood leaves those pairs unpushed. The case: the operator lifts the advised hub pause while one hub's sweep failed and awaits its re-run.
   - D6 says nothing about this.
   - Evidence: `internal/tenderengine/doc.go:74-77`; `internal/tendercli/deps.go:128-132` (`Owners` compares against `machinename.Name`).
   - Fix: D6 advises lifting the hub pause only after that hub's sweep reports done, or names the push that recovers a skipped pair.

**Tool calls**

1. grep for readers of recorded machine names and of `FormatMachineAddress`: the context lacked their non-owner-set readers.
2. grep of the dobby engine and dobbydeps for where `Machine` is resolved: the context did not show whether dobby reads the name once or per check.
3. grep for `PauseOwnedElsewhere`/`EndOwnedElsewhere` callers and dobby's owner-pause lift: the context named neither the callers nor the lift.
4. Read of `internal/tendercli/deps.go:225-294`: to confirm that `PauseOwnedElsewhere` ends processes, which `tendercli/doc.go` stated only for the pair of calls.
