```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Per decision**
- **D1:** case `osl-1046` with `{"name":"osl46"}` at `$XDG_CONFIG_HOME/lym/machine.json`. The resolver finds the file and the reader returns `osl46`. Purpose met.
- **D2:** `standalonestate.UserConfigDir`, then `githubclient.CredentialDir`, then the Slack paths through `hubgeom.SlackMachine`. The directory is unchanged. Purpose met. The return shapes differ: `UserStateDir` returns an error and `CredentialDir` returns a bool, so the delegation needs an adapter. That is plan-level.
- **D3:** case a corrupt file. `Name` errors at every owner check, so start, resume and landing all fail loudly with the path. Purpose met.
- **D4:** case `OSL46`. The verb refuses it and the reader errors on it. Purpose met.
- **D5:** case `lym machine name osl46` on `osl-1046`, which has no file. The collision read finds `prime_owner: osl-1046`, which is the current name, so it passes. The file gets `name: osl46, former_names: [osl-1046]`. The sweep follows. Purpose met; see finding 3.
- **D6:** case prime `osl-1046` plus beam `b1` owned by `osl-1046`. `ReadOwners` reads the local Bolt, then `SetOwners(Machine=osl46, Slugs=[b1], Prime, replace={osl-1046})` runs. Purpose met only if the replace field implies takeover; see finding 2. The window bound breaks for Finalize; see finding 1.
- **D7:** case a Land entry with `Machine: osl-1046`, and the dobby asks with `osl46`. With the name set passed in, the landing is listed. Purpose met. Only one caller passes the machine: `internal/beamcli/dobbydeps.go:192`.
- **D8:** case a package test calling `machinename.Name` under `tmuxkit.Main`. `XDG_CONFIG_HOME` points at the run directory, which holds no file, so the hostname rule applies. Purpose met; see finding 4.

**Findings**

1. **MEDIUM** (D6 race bound, Open risks): the bound's way forward is wrong for a landing caught in the window.
   - The record says the run "takes an owner pause" and that `lym tender resume <slug>` resumes it.
   - A Finalize that crosses the window halts with `owner check:`. Its run is blocked with a reason opening `landingshed.OwnerHaltPrefix`; it is not paused.
   - `internal/adoptcli/doc.go` (green end, step 4) clears that state with `lym loom resume` in the pair, not with `lym tender resume`.
   - The bound and the verb's output should name both ways forward: `lym tender resume <slug>` for an owner pause, and `lym loom resume` in the pair for an owner halt.

2. **MEDIUM** (D6): the record does not say how the new replace field relates to `Takeover`.
   - `SetOwners` without `Takeover` leaves alone any entry whose effective owner reads as another machine.
   - Here `osl-1046` differs from `req.Machine` (`osl46`), so it reads as another machine and nothing would be written. The sweep would do nothing and report every slug in `Lost`.
   - D6 should state that the replace set allows the overwrite on its own (takeover limited to the set), or that the sweep also sets `Takeover`.

3. **LOW** (D5): the collision check and `ReadOwners` read the local Bolt before `SetOwners` pulls.
   - A beam the other machine adopted on origin may still read `osl-1046` locally.
   - The replace set guards the write, because the pulled owner `neve` is outside the set and goes to `Lost`. The write is safe.
   - The collision check can miss an owner that only origin holds. The record should state this as a bound, or say the check reads after a pull.

4. **LOW** (D8): setting `XDG_CONFIG_HOME` for every test also moves where git and tmux look for their XDG config (`$XDG_CONFIG_HOME/git/config`, `$XDG_CONFIG_HOME/tmux/tmux.conf`).
   - A git config that a test unknowingly relied on stops being read.
   - `internal/gitkit/hermetic.go` exists, so hermetic git probably covers this. The record should state the side effect or note that it is safe.

5. **LOW** (D5, `PATTERN-lym-files`): the record does not name the lock path for the machine file.
   - `state.WriteJSON` and `state.UpdateJSON` take an explicit `lockPath`.
   - The file is per-user and outside any hub, like `hubs.json`, whose lock `hubs.lock` sits beside it.
   - Saying the lock sits beside the file, as `hubregistry` does, would close this; the tracked-file lock rule does not apply.

**Tool calls**
- Read `setTempEnv` and grepped testkit for git-config isolation and the `state` write API. My context had no body for `setTempEnv` and no signatures for `internal/state`.
- Grepped for hermetic git in `gitkit` and `gitexec`. My context did not show whether test git ignores the user's XDG config.
