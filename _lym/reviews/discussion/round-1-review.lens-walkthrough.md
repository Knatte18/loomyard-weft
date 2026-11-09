```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision end to end on one concrete case and checked the end state against the Decision's own purpose.

**Decisions walked**
- **D1** (case: `machine.json` holding `osl46` in the default config directory): purpose met. The case with a relative `XDG_CONFIG_HOME` fails; see F4.
- **D2** (case: `githubclient.CredentialDir` delegating to `standalonestate.UserConfigDir`, environment unchanged): purpose met for the GitHub and Slack paths. A relative `XDG_CONFIG_HOME` breaks D1's purpose; see F4.
- **D3** (case: no file, then a file with `osl46`, then a file that does not decode): purpose met for the reader.
- **D4** (case: the name `osl.46` given to the verb, then to the reader): purpose met.
- **D5** (case: rename `osl-1046` to `osl46`, then repair a bad file over a machine that was already swept to `osl46`): rename met. Repair not met; see F2.
- **D6** (case: hub H, whose local Bolt has beam `b1` owned by `osl-1046` and whose origin also has beam `b2` owned by `osl-1046`): not met; see F1.
- **D7** (case: a landing finished under `osl-1046`, then the dobby asks for landings after the rename): not met; see F3.
- **D8** (case: the acceptance criterion "`lym machine name osl46` writes `osl46`" run as a `machinecli` test on Linux): not met; see F5.

**Findings**

F1 — MEDIUM, design. Location: D6, sweep slug selection.
The slugs to sweep come from `boardengine.ReadOwners` on the hub's local Bolt, before `SetOwners` pulls.
So a beam or prime owned by a former name that only origin holds is never put in `Slugs`, and the pull does not add it.
A hub whose local Bolt holds no former-name owner is reported as "nothing to sweep", makes no `SetOwners` call and never pulls at all.
The verb exits 0 while origin still records the old name, and that owner reads as another machine's.
D5's bound mentions the pre-pull local read for the collision check only; the sweep's own selection has the same gap and no bound.
- Evidence: `internal/boardengine/ownerset.go:294-308`; the local read is `ReadOwners`, and the pull is inside `SetOwners` at `ownerset.go:112-113`.
- Fix: pull each hub's Bolt before selecting its slugs, or have the write select under its own pull. Alternatively, state the bound and the way forward.

F2 — MEDIUM, design. Location: D5, the bad-file repair together with the collision check.
Case:
1. The verb swept owners to `osl46` while the file held `name: osl46` and `former_names: [osl-1046]`.
2. A hand edit breaks the file. Either it no longer decodes, or `name` becomes invalid.
3. The operator runs D3's way forward, `lym machine name osl46`.

Walking it through:
- The repair drops the bad name, or loses the whole file when it does not decode. `osl46` is now neither the current name nor a former name.
- The collision check finds `osl46` recorded as an owner on the swept boards, and it is not this machine's current or former name.
- So the verb refuses it as another machine's name.

D3 names `lym machine name &lt;name&gt;` as the repair, yet restoring the machine's own previous name is refused.
Choosing any other name leaves every `osl46` owner unswept, because `osl46` is not in `former_names`.
D5's claim that no owner can hold the dropped name holds only for the invalid string itself, never for the valid name the hand edit replaced.
- Evidence: D5 lines 83 and 89-92 of `decision-record.md`.
- Fix: decide how the repair recovers the last valid name, or give the collision check a bounded exemption for it. Otherwise state the case and its way forward.

F3 — MEDIUM, design. Location: D7 against D3.
`DobbyLandings` is reached through `machineLandings(machinename.Name, func(machine string) …)`, which hands it one name.
The record gives it no source for the former names:
- D3 defines only `machinename.Name`, which returns the current name.
- D5's own reading of the raw file is the verb's alone.
- D3 makes `internal/machinename` the only reader of the machine file.

D7's input, the former names, is produced by no step, so its mechanism cannot run as written.
- Evidence: `internal/beamcli/dobbydeps.go:190-192`; `battencli.DobbyLandings` takes `machine string` at `internal/battencli/dobbyreads.go:262`.
- Fix: add to D3 a `machinename` reader that returns the former names, and route `machineLandings` and `DobbyLandings` through it.

F4 — LOW, design. Location: D2 against D1's purpose.
Keeping `CredentialDir`'s rule takes a relative `XDG_CONFIG_HOME` as given.
Case with `XDG_CONFIG_HOME=cfg`:
- The verb, run from hub A, writes `A/cfg/lym/machine.json` and `machine.lock`.
- The dobby of hub B, whose working directory is B, reads `B/cfg/lym/machine.json`. No file is there, so it falls back to the hostname rule.

The end state misses D1's rationale of one name for every hub on the machine.
It also writes per-user state into whatever directory the verb runs in.
D1's bound covers a different `XDG_CONFIG_HOME` per process, not the same relative value resolving differently per working directory.
- Evidence: `internal/githubclient/cache.go:49-50`; `internal/standalonestate/standalonestate.go:154-159`, which ignores a relative base for state for exactly this reason.
- Fix: ignore a relative `XDG_CONFIG_HOME` for the machine file, or state the bound.

F5 — MEDIUM, design. Location: D8 against D5's acceptance criteria.
D8 isolates only `internal/machinename`'s own tests. On Linux the test environment sets `XDG_STATE_HOME` and leaves `XDG_CONFIG_HOME` and `HOME` alone.
A `machinecli` test of the criterion "`lym machine name osl46` … writes the name `osl46`" therefore:
- writes the developer's real `$HOME/.config/lym/machine.json`, which renames the developer's machine;
- has every other lym process on that machine pick up the new name.

The hub registry is isolated through `XDG_STATE_HOME`, but the file write is not. D8's claim that no test outside the package needs a change holds only for readers, not for the verb's writer.
- Evidence: `internal/testkit/tmuxkit/tmuxkit.go:604-610`. `LOCALAPPDATA` is isolated only on Windows.
- Fix: give the verb a seam for the file path, or bring `machinecli` into D8's isolation, and say so in D8.

**Tool calls**
- Bash: `internal/beamcli/dobbydeps.go:170-200` and the env lines of `setTempEnv` in `internal/testkit/tmuxkit/tmuxkit.go`. My context had only `DobbyLandings`' call line, not how its machine argument is sourced, and lacked which variables the test environment isolates.
