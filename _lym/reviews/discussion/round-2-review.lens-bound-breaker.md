```yaml
lens: bound-breaker
outcome: reported
```

I attacked every bound and claim in the decision record. Three claims break, all MEDIUM, and five more findings are LOW. The rest hold against the code I read.

**Claims attacked**
- D1, the file lives "beside `slack.json` and `credentials.json`": **broken** when `XDG_CONFIG_HOME` is relative (F6).
- D1 bound, each process reads the file its own environment resolves: held.
- D2, `githubclient.CredentialDir` keeps its rule by delegating: held.
- D2, the relative-variable rule mirrors `standalonestate.UserStateDir`: held.
- D3, a bad file is an error and never a fallback, because "a wrong name silently changes which entries read as owned here": **broken** (F2).
- D3 bound, a caller that reads the name per check takes the new name at its next check: held. The dobby's `Machine`, `Owner` and `recheckOwnerPause` seams, `orchcli` `beamOwnerElsewhere`, `fronttask` `OwnedHere`, and beamcli create/land all call `machinename.Name` per call.
- D3/D5, `names` is decoded with no validation and its other entries are the former names: **broken** (F1).
- D5 bound, the verb writes only owners recorded under this machine's former names: **broken** (F1).
- D5 bound, the collision check misses only the pull race and a hub whose pull failed: **broken** (F3).
- D5 bound, a reused former name passes the check, with its way forward: held.
- D5, a run with the name already set re-runs the sweep: held, since the replace set is the former names.
- D6, the write selects owners after its own pull: held for the normal path (`internal/fabricengine/bolt.go:224-239`). It breaks under `BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH` (F4).
- D6, a done entry keeps its owner: held (`internal/boardengine/ownerset.go:188`, `isDone` skip).
- D6, a hub whose write records nothing is reported as having nothing to sweep: held, since `writeOwners` returns nil when nothing is written. But a sweep write that loses to a competing origin write comes back in `OwnerWrite.Lost`, and D6 does not say how that is reported (F5).
- D6 window bound: the orch pauses at bring-up, the dobby holds only under `pauserecord.HoldCaller`, and a lift skips pairs it reads as owned elsewhere. Held (`internal/orchcli/doc.go:203-215`, `internal/dobbyengine/doc.go:155-176`, `internal/tendercli/doc.go:43-60`).
- D6, the dobby's probe lifts its own owner pause after the sweep: held (`internal/dobbyengine/owned.go:85-100`, read per call).
- D7, shed history `Machine` is the only persisted name besides owners: held. `shedengine.HistoryEntry.Machine` (`internal/shedengine/status.go:120`) is the only `json:"machine"` field in `internal`.
- D8, other packages' tests read the developer's file and pin names through seams: **broken** on Windows (F7). On Linux it holds, because the integration tests that call `machinename.Name()` compare against the same file the code under test reads.
- Constraints, every owner comparison is string equality: held (`internal/boardengine/owners.go:175-177`).
- Constraints, the Slack `retireEntry` and `name_taken` claims: held (`internal/slackengine/keeper.go:422-453`, `internal/slackengine/channelname.go:29-44`).
- `PATTERN-leaf-packages`, only stdlib plus `internal/standalonestate`: held. D3 has the leaf decode raw, and the writer sits in `internal/machinecli`.

**Findings**

**F1 — MEDIUM, design.** D3 (`machinename.ReadFile`, `machinename.Names`), D5's first bound, and D7.
- **Issue:** Only `name` is validated. `names` is never checked against D4, yet D5 sends the operator to hand-edit it. A `names` holding `""` puts `""` in the replace set.
- **Effect on owners:** D6 selects owners whose recorded owner is in the set, so `""` selects every beam with no owner and a hub with no prime owner, and writes this machine onto them. That breaks "never writes an owner recorded under any other name". An unowned prime means no machine owns it, by `internal/boardengine/owners.go:176` (`owner != ""`).
- **Effect on landings:** D7 would count every Land entry whose `Machine` is empty as this machine's. Entries written before the field existed decode empty (`internal/shedengine/status.go:118-120`).
- **Fix:** validate each entry of `names` with D4's check in the reader and the verb, and refuse or drop empty and invalid ones.

**F2 — MEDIUM, design.** D3's rationale for no fallback.
- **Issue:** D3 says an error is safer than a hostname fallback. But the dobby's beam owner check treats a name error as "not owned elsewhere" and goes on (`internal/dobbyengine/loop.go:553-571`).
- **Effect:** a hand-broken file makes this machine's dobby drive a beam another machine owns, which is exactly the wrong-ownership harm D3 means to prevent. Before this change a name error needed an unreadable hostname; with the file it is an ordinary case.
- **Fix:** either fail closed in readers that treat a name error as permission to act, or state the bound and the readers' actual behaviour.

**F3 — MEDIUM, design.** D5's collision-check bounds.
- **Issue:** The check sees only owners recorded on the hubs this machine sweeps. Take a live machine whose hostname-rule name is `neve` that has recorded no owner on those boards yet: it owns nothing, or works only on hubs this machine does not have. `lym machine name neve` passes.
- **Effect:** once that machine records an owner, both machines read each other's entries as their own (`internal/boardengine/owners.go:176`).
- **Gap:** D5's shared-name bound says "by that case or the one above", which covers only the pull race and a reused former name. This case is neither named nor bounded.
- **Fix:** add it to the bound, with the same way forward.

**F4 — LOW, design.** D5's pull before the check, and D6's "selects, after its pull".
- **Issue:** `SetOwners` under `BOARD_SKIP_GIT` or `BOARD_SKIP_PUSH` writes with no pull (`internal/boardengine/ownerset.go:79,107-110`), and `Bolt.Pull` with `SkipGit` returns at once (`internal/fabricengine/bolt.go:280-282`).
- **Effect:** under either variable, the claims that the sweep catches "an owner only origin holds" and that the check does too are false. Neither bound names this.

**F5 — LOW, consistency.** D6's per-hub reporting.
- **Issue:** a competing write that origin holds wins over the sweep's replay. `SetOwners` then reports the slug in `OwnerWrite.Lost`, not as an error (`internal/boardengine/ownerset.go:155-160`).
- **Gap:** D6 reports only "write fails" and "records nothing", so a partial sweep with lost entries has no stated output and no way forward.

**F6 — LOW, consistency.** D1's "beside `slack.json` and `credentials.json`" against D2's split.
- **Issue:** with a relative `XDG_CONFIG_HOME`, `githubclient.CredentialDir` resolves `&lt;relative&gt;/lym`. The machine file resolves through `standalonestate.AbsoluteUserConfigDir` to `$HOME/.config/lym` (`internal/githubclient/cache.go:48-50`), so the two are in different directories.
- **Gap:** D1 states "same directory" without that exception.

**F7 — LOW, consistency.** D8 ("No test environment variable changes"; other packages' tests "only read the developer's file").
- **Issue:** on Windows, `tmuxkit.Main` already points `LOCALAPPDATA` at the test's temp dir (`internal/testkit/tmuxkit/tmuxkit.go:600-611`). So those tests read no machine file and get the hostname name, not the developer's file.
- **Gap:** D8's claim and its bound about a developer's bad file hold only off Windows.

**Tool calls**
1. Grepped six long-lived callers (beamcli `dobbydeps.go`, hubgeom `dobbygeom.go`, orchcli `watch.go` and `start.go`, fronttask `hub.go`, beamcli `deps.go`) for `machinename.Name`: the context lacked how long-lived processes resolve the name, which D3's per-check bound depends on.
2. Grepped `dobbyengine` and `OwnedElsewhere` for where `deps.Machine()` is called: the context lacked whether the dobby caches the name.
3. Read `dobbyengine/loop.go:545-575`, `owned.go:85-100`, the out-of-package tests that read the name, and tmuxkit's env setup: the context lacked the dobby's handling of a name error and the test environment behind D8.
4. Grepped those test lines and `tmuxkit.setTempEnv`: needed to judge D8's claim about other packages' tests.
5. Grepped `internal/state` for its write and read functions: wanted to know whether the writer is atomic, since the leaf reader cannot take the lock. Inconclusive, so no finding on it.
