```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function or branch the diff adds or changes, step by step, with concrete inputs.

**Per function or branch: the case I ran, correct or not**

- `orchengine.LoadConfig` remote-control checks:
  - template default: correct.
  - `remote_control: false` with an empty name: loads, correct.
  - on with `" "`: refused, correct.
  - a name holding a tab: refused, correct.
  - on with `{slug}` only: loads and renders literally, correct and documented.
- `Config.RemoteControlSessionName`:
  - `{name}` with name `ly:orch`: one pass, no refill, correct.
  - an empty machine name gives `::ly:orch`. Not reachable, because `machineName` errors first. Correct.
- `orchCLI.orchHasRemoteControl`:
  - `ModelChoice.Engine` is always empty, so `Lookup("")` returns the default engine. That is the same engine `Start` resolves for the same spec. Correct.
  - a `Lookup` error gives false. Correct.
- `remoteControlSessionName`:
  - a listed name that fails to parse is wrapped and returned. Correct.
  - a `machineName` error is wrapped. Correct.
- `launchRemoteControlName`:
  - switch off gives `""`. Correct.
  - Codex default engine gives `""`. Correct.
  - `hasStrand=false` with no shortname record: logs a Warn and returns `""`. Correct.
  - `hasStrand=true`: uses `listed.Name`. The relaunch's `NameOverride` "orch" is never renumbered, so the new strand gets the same name. Correct.
- `startOrch` relaunch: `launchOrch` fails, so nothing is recorded. Correct.
  - Success writes the record before `saveFreshLaunch`. If `saveFreshLaunch` then fails, the record names the new guid while the state may keep the old one, so the button reads dim. Display only; LOW at most, not filed.
- `startOrch` StartSpawnWatcher and already-running branches write no record. See F2.
- `launchOrch` resume fails and falls back fresh: both specs carry the name. Correct.
- `buildLaunchCmd` / `buildResumeCmd`:
  - the name `ly:orch[neve]` with a `'` in it: posix renders `'ly:orch[neve]'` with `'\''` for the quote; pwsh doubles the quote.
  - Either way it is one argument, and it comes after every positional argument. Correct.
  - An empty name adds nothing. Correct.
- `Runner.TurnOnRemoteControl`:
  - a name holding `\n`: refused. Correct.
  - Codex: the sequence is nil and `sendSequence` returns nil at once (`send.go:688`), so nothing is typed. Correct.
  - Claude: a verified send with `startsNoTurn`. Correct as far as lym's side goes. I did not verify Claude's UI after `/remote-control` (see F4).
- `Claude.RemoteControlSequence`: correct.
- `creelengine.Command.validate` for remote-control: an empty argument and one holding a control character are both refused. Correct.
- `creelcli.runSessionCommand` remote-control case: correct.
- `orchroute.FixedStrandName` / `StrandOf` refactor: same results and same errors as before. Correct.
- `orchengine.WriteRemoteControl` / `RemoteControlOn`:
  - an absent file reads false.
  - another strand's record reads false.
  - garbage in the file is an error.
  - Correct.
- `remoteControlBarWord` / `remoteControlBarWordAt`:
  - an empty dir gives `""`.
  - a dir that does not resolve gives `""`.
  - no state gives `""`.
  - a state with a strand but an unreadable record gives `rc-off`.
  - Correct.
- `turnOnRemoteControl`:
  - two orch strands: misreported (F1).
  - inject succeeds but the record write fails: misreported (F3).
  - no live strand, a Codex orch, an inject failure: all correct.
- `remoteControlOnCmd` RunE: on error, `ShowToggleOutcome` shows the message before the error envelope is printed. Correct.
- `remoteControlBarCmd` with `PersistentPreRunE` skipped for `rc-bar`: no `c` field is used. Correct.
- `bindingLinesLocked`:
  - socket unresolved: every line empty.
  - test binary: Slack and RC lines empty, switch lines kept.
  - switch lines failing still returns the toggle lines, as before.
  - Correct.
- `bindingArgvs` RC branch: appended only when its line is pinned. Correct.
- Bar formats: the RC button is shown only on `rc-on`/`rc-off` and lit only on `rc-on`. Correct.
- `composeRemoteControlClickCommand` / `composeRemoteControlStateCommand`: `#` is escaped, and the test binary is refused. Correct.

**Findings**

- **F1 LOW, design.** `internal/orchcli/remotecontrol.go:297`. When the worktree holds two orch strands (`len(named) > 1`), `turnOnRemoteControl` refuses with "this worktree runs no live orch strand; way forward: run `lym beam start`". That misreports the cause. `startOrch`'s own check (`start.go:613-619`) names the duplicate strands and points to `lym reed remove`.
- **F2 LOW, design.** `internal/orchcli/start.go:684-697`, read with `remotecontrol.go:366-374`. The StartSpawnWatcher and already-running branches adopt a listed strand into `st.Strand` (`adoptListedStrand`) but write no remote-control record. A live orch whose guid differs from the record's therefore reads dim even if it was launched with `--remote-control`. The record and the state can disagree after an adopt.
- **F3 LOW, design.** `internal/orchcli/remotecontrol.go:305-310`. When the inject is queued but `WriteRemoteControl` fails, `rc-on` reports a refusal on the bar while the `/remote-control` command still gets typed. The click's error says it failed although remote control will turn on, and the button stays dim.
- **F4 NIT, design, unverified.** `internal/shuttleengine/session.go:1224-1230` and `internal/shuttleengine/send.go:691-695`. The command goes through `sendVerified` with `startsNoTurn`. If Claude Code answers `/remote-control` with a panel and no input box, the verified send may read it as not landed, and creel would requeue and type it again. I could not check Claude Code's UI, so this rests on that assumption and is not a confirmed defect.

**Tool calls**

1. `grep` of `EngineSet.Lookup` and `ModelChoice`: to check how `orchHasRemoteControl` resolves an empty engine name, which my context lacked.
2. `grep` of `sendSequence` and `GoClient.InjectCommand`: to check what a nil sequence does and what a zero TTL means. Neither was in my context.
3. `sed` of `claudeengine/command.go` 410–500 and a grep of the builders' callers: to check whether another launch or fallback line skips the flag. Only `OpenLaunch` does, and it serves opened sessions, not the orch.
