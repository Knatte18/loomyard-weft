```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims and bounds, looking for inputs, interleavings or states that break them.

**Findings**

1. **MEDIUM, D6 (race bound).** The way forward `lym tender resume <slug>` does not cover a run that halts in Finalize.
   - Counterexample: a run's landing reaches Finalize's owner check between the file write and its hub's sweep. Finalize halts it as blocked with the reason prefix `landingshed.OwnerHaltPrefix` (`owner check: `), which is a halt, not a pause.
   - The adopt doc's green end, step 4, recovers this case with `lym loom resume` in the pair, not with `lym tender resume`.
   - So the bound and Open risks understate the outcome. The verb's output should also name `lym loom resume` in the pair for an owner-check halt.

2. **MEDIUM, D3 with D5 (way forward loops).** D3 names `lym machine name <name>` as the way forward from a bad file, but D5 starts by reading the current name to fill `former_names`.
   - Counterexample: a hand-edited file holds `{"name":"OSL46"}`. Every `machinename.Name` call now errors, including the verb's own read of the old name.
   - If the verb refuses, the way forward loops. If it proceeds, the record does not say which old name it records or whether it keeps the existing `former_names`.
   - The record should state that the verb proceeds over an invalid or undecodable file and keeps whatever `former_names` decodes.

3. **MEDIUM, D1 and D3 (one name per machine).** The claim that the name is "one value for every hub on the machine" breaks when processes see different `XDG_CONFIG_HOME` values.
   - Counterexample: the operator's shell exports `XDG_CONFIG_HOME=/x` and runs `lym machine name osl46`, which writes `/x/lym/machine.json`. A dobby or tmux server started without that variable reads `~/.config/lym`, finds no file and falls back to `osl-1046`. It then reads every swept owner as another machine's.
   - The record is silent on this. It needs at least a bound, or a Constraint that the directory resolves the same way in every lym process.

4. **LOW, D5 and D7 (bound "never writes an owner recorded under any other name").** The bound holds for the current name, but a former name can later belong to another machine.
   - Counterexample: osl-1046 renames to `osl46`. A reinstalled host named `osl-1046`, with no file, then records owners under `osl-1046`. A re-run of `lym machine name osl46` sweeps those owners as this machine's former name.
   - The same collision makes D7 count that machine's Land entries as this machine's.
   - The D5 collision check covers only the new name, not the former names.

5. **LOW, D3 ("a running dobby or orch takes a new name at its next owner check").** This holds for the per-check callers, but some long-lived processes resolve the name once.
   - `hubgeom.SlackGeometry` (`slackgeom.go:58`, `Machine: machine`) fills it once per relay start. A running Slack relay keeps the old name, and creates new channels under it, until it restarts.
   - The Slack constraint does not mention this. The outcome is harmless by the retire path, but the claim is broader than the code supports.

6. **LOW, D5 ("writes the file through `internal/state`").** `state.WriteJSON` and `state.UpdateJSON` take a `lockPath`. `PATTERN-lym-files` routes lock paths through `lock`, whose helpers cover `_lym` and Bolt files, not a per-user file.
   - The record names no lock for `machine.json`. Precedent: `slack-socket.lock` sits in the same directory.
   - This is a design gap only if the plan reaches for `lock.TrackedFileLockPath`.

7. **LOW, D8.** Setting `XDG_CONFIG_HOME` in `setTempEnv` also moves what git (`$XDG_CONFIG_HOME/git/config`), tmux (`$XDG_CONFIG_HOME/tmux/tmux.conf`) and `vscode/keybindings.go` read in every test package.
   - That is more hermetic, not less, but it is an unstated side effect on every package that uses `tmuxkit.Main`.

8. **LOW, D2 acceptance criterion ("same directory for the same environment").** `standalonestate.UserStateDir` ignores a relative `XDG_STATE_HOME`, while `githubclient`'s `cacheDir` accepts a relative `XDG_CONFIG_HOME`.
   - If `UserConfigDir` copies the state-dir rule, `CredentialDir`'s result changes for a relative value. The record should say which rule wins.

**Claims attacked**

- D6, owner comparison by string equality: held. The equality sites I grepped (`dobbyengine` start, loop and owned; `battencli` cancel; `fabriccli` anchordeps; `beamcli` create; `boardengine.OwnedElsewhere`) all read the current name, so they agree once the sweep has run.
- D7, the only comparison of a shed history `Machine` is in `DobbyLandings`: held. The grep found no other `entry.Machine ==` site.
- D6, the sweep reads owners from the local Bolt before `SetOwners` pulls: held. Only this machine writes owners under its old name, so the local Bolt already holds them.
- D6, a concurrent `lym adopt` during the window: held. Adopt reads the new name after the file write.
- Constraints, the Slack retire path: held. A channel made under the old name is archived without the dated rename.
- D6 race bound: broken (finding 1).
- D3 and D5 bad-file way forward: broken (finding 2).
- D1, one name per machine: broken (finding 3).
- D5, "never writes another name's owner": broken by a reused former name (finding 4).
- D3, "running processes take the new name": partly broken (finding 5).
- D2, same directory: at risk (finding 8).
- D8, isolation: held, with a side effect (finding 7).

**Tool calls**

- Grepped the machine and owner equality comparisons in production code, to find comparison sites beyond `OwnedElsewhere` and `DobbyLandings` that my context lacked.
- Read `tmuxkit.setTempEnv` and grepped the `XDG_CONFIG_HOME` readers, to test D8 and D1 against the real environment handling.
- Grepped the `internal/state` API, `OwnerHaltPrefix` and the `Machine` field in `slackgeom.go`, to test D5's write path, the D6 halt-versus-pause bound and D3's every-call claim.
