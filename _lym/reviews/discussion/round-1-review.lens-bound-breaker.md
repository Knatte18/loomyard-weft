```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break each bound, guarantee and claim about existing code in the decision record. Six claims broke and the rest held.

**Claims attacked**
- D1, the per-user config directory is where `githubclient.CredentialDir` resolves (`internal/githubclient/cache.go:40-56`): held.
- D1, `machine.lock` sits beside the file as `hubs.lock` sits beside `hubs.json` (`internal/hubregistry/hubregistry.go:20,23`): held.
- D1, the name is one value for every hub, and only a process with another `XDG_CONFIG_HOME` reads another file: **broken** (F2).
- D2, `internal/standalonestate` imports only the standard library (`internal/standalonestate/standalonestate.go:6-15`): held.
- D2, a non-empty `XDG_CONFIG_HOME` is taken as given (`cache.go:49-50`): held as a fact. It feeds F2.
- D3, every owner check reads the name per check: held. The callers call `machinename.Name` inside their closures (`internal/orchcli/watch.go:60`, `internal/battencli/dobbyreads.go:326`, `internal/loomcli/pawlwiring.go:92`, `internal/tendercli/deps.go:129`), and D7's landings go through `machineLandings(machinename.Name, …)` (`internal/beamcli/dobbydeps.go:189-191`).
- D3, the Slack relay fills its geometry once (`internal/hubgeom/slackgeom.go:50`): held.
- D3, `internal/machinename` becomes the only reader of the machine file: **broken** by D5 (F6).
- D5, the verb never writes an owner recorded under a name outside this machine's former names: held as written. `writeOwners` only overwrites when the replace set allows it.
- D5, the collision check refuses another machine's name: **broken** by the former-name exemption (F3).
- D6, after a sweep every beam owner and the prime owner equal to the old name reads the new name: **broken** for done beams (F4).
- D6, a hub pause (`lym tender pause`) keeps the window's cases from arising: **broken** (F1).
- D6, `Takeover` stays unset and only `lym adopt` passes it (`internal/adoptcli/adopt.go:371`, `internal/boardengine/doc.go:177`): held.
- Constraints, every owner comparison is string equality (`boardengine.OwnedElsewhere`, `internal/boardengine/owners.go:175-177`): held.
- Constraints, a rename is safe for Slack: an existing channel keeps working by id and is archived under its old name. This holds as stated, but **broken** if the machine later goes back to the old name (F5).
- D7, `DobbyLandings` takes the name per call and matches `entry.Machine` exactly (`internal/battencli/dobbyreads.go:262-276`): held.
- D8, tests outside the package read the developer's file as lym does: held. Those tests take the name from `machinename.Name` themselves (`internal/hubgeom/hubgeom_test.go:520`, `internal/fronttask/hub_integration_test.go:88`, `internal/beamcli/create_integration_test.go:46`).

**Findings**

- **F1 — MEDIUM — design — D6, window bound.** The recommended hub pause does not hold the owner checks the window reaches.
  - A pause "never touches the orch" (`internal/tenderengine/doc.go:42`).
  - The dobby holds a run only for "a live pause entry of the run's own scope carries pauserecord.HoldCaller" (`internal/dobbyengine/passes.go:47`). Its owned-elsewhere probe goes ahead unless `run.Held` (`passes.go:191`).
  - Counterexample: the operator runs `lym tender pause` with no slug, then `lym machine name osl46`. Before the sweep reaches the hub, the dobby probe reads a beam run as owned elsewhere. It calls `PauseOwned`, which is `tendercli.PauseOwnedElsewhere` (`internal/beamcli/dobbydeps.go:156`). Then `EndOwnedElsewhere` ends the run's processes (`internal/tendercli/doc.go:45-46`).
  - The support log admits the fact was never established (`support-log.md:39,53`). The advice still tells the operator the pause prevents these cases. This is research left undone.

- **F2 — MEDIUM — design — D1 and D2.** A relative `XDG_CONFIG_HOME` makes the machine file depend on the working directory.
  - Counterexample: `XDG_CONFIG_HOME=.config` is set for every process. The operator runs `lym machine name osl46` in the prime, which writes `&lt;prime&gt;/.config/lym/machine.json` and sweeps every hub to `osl46`.
  - The dobby and orch run with another working directory. They find no file, fall back to the hostname rule and get `osl-1046`. Every swept owner then reads as another machine's.
  - D1's bound covers only a process with a different environment, and the path the verb prints is relative too.
  - Evidence: `cache.go:49-50` and `standalonestate.go:35-40`. The latter explains why the state directory refuses a relative base.

- **F3 — LOW — design — D5, collision check.** The check lets through any name that is one of this machine's former names, even when another host now holds it.
  - Counterexample: machine A renames from `a1` to `a2`. Host B later takes the name `a1` and owns beam P. A runs `lym machine name a1`. The check sees owner `a1` on P but lets it pass as A's former name.
  - From then on every owner check on A reads P as owned here, so A can start, resume and land B's runs.
  - D5's bound names only the sweep and D7 for this case, not the collision check or the owner checks.
  - Evidence: `decision-record.md:83,96-97`.

- **F4 — LOW — consistency — D6 and its acceptance criterion.** The sweep's `SetOwners` call skips done entries: `if index &lt; 0 || isDone(...) ... continue` (`internal/boardengine/ownerset.go:188`).
  - A done beam project recorded under the former name keeps that owner.
  - This contradicts the criterion "every beam owner and the prime owner equal to the old name … reads the new name" (`decision-record.md:167`).

- **F5 — LOW — design — Constraints, Slack.** After the relay restarts, `retireEntry` builds the channel's base name from the new machine name (`internal/slackengine/keeper.go:423`).
  - An old channel's name no longer equals that base name, so it skips the dated rename and is archived under its undated old name (`keeper.go:424-437`).
  - If the machine later goes back to the old name, a new channel for the same slug hits `name_taken`. `adopt` then refuses because the channel is archived (`keeper.go:175-177`).
  - So "a rename is safe there" does not hold for a rename back to a former name.

- **F6 — LOW — consistency — D3 against D5 and the Notes.** D3 says `internal/machinename` "becomes the only reader of the machine file" (`decision-record.md:63`).
  - D5's repair path "reads the file itself instead of through `machinename.Name`" (`decision-record.md:89`). That verb lives in `internal/machinecli`, and it also writes the file.
  - The Notes ask the sole-source scan to pin that no other production file names the machine file (`decision-record.md:186`). Unless the path and the raw read come from `internal/machinename`, that scan fails on the verb.

**Tool calls**
1. `internal/beamcli/dobbydeps.go:140-200`, plus the `FormatMachineAddress` callers and the tests that call `machinename.Name`. My context did not show whether D7's machine is resolved per call, or which tests read the name.
2. The `machinename` uses in three of those tests. My context did not show whether they compare against the hostname rule, which D8's claim depends on.
