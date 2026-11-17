```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes, using one concrete input per case.

**Cases run**
- `turnOnRemoteControl`, pane `{%1, "tst:orch", off}`: the name parses, the command is queued, the pane is marked, and the fields are returned. Correct.
- `turnOnRemoteControl`, pane with empty `StrandName`: it refuses, sends nothing and marks nothing. Correct.
- `turnOnRemoteControl`, `RemoteControlOn: true`: it refuses and sends nothing. Correct.
- `turnOnRemoteControl`, name that does not parse: it returns the parse error, sends nothing and marks nothing. Correct.
- `turnOnRemoteControl`, inject fails: it returns a wrapped error with a way forward and marks nothing. Correct.
- `turnOnRemoteControl`, mark fails: it logs a Warn and returns the fields with a warning. Nothing is left inconsistent beyond the documented dim button. Correct.
- `rc-on` RunE, `ClientAgentPane` fails: the error reaches `ShowToggleOutcome` and `output.Err`. Correct.
- `rc-on` RunE, session whose `@lym_button_dir` is empty: wrong. The pinned line becomes `cd  &gt;/dev/null…; lym orch rc-on …`. The posix `cd` goes to `$HOME`, and `PersistentPreRunE` then fails in `lymcwd.Resolve`/`wireCLI`. It aborts before RunE, so `ShowToggleOutcome` never runs and the line's output is discarded. The click fails silently (finding 1).
- `clientAgentPaneVia`: three fields, two fields, an empty pane id, empty output and a tmux error. All handled. Correct.
- `markPaneRemoteControlOnVia`, set-option fails: the error is wrapped and names the pane. Correct.
- `Engine.MarkRemoteControlOn`, unknown guid: `resolveLivePaneID` errors and nothing is set. Live pane in this session: `set-option -p`. Correct. It runs after the runner's start returns, so it takes no nested op lock.
- `markLaunchedRemoteControl`, nil seam: returns. Mark error: logged at Warn and the launch is not failed. Correct.
- `startOrch`, empty `remoteControlName` (switch off, engine without RC, or a name that cannot be formed): nothing is marked. Non-empty: the new guid is marked. Correct.
- `startOrch`, strand already running or watcher-only path: nothing is marked, and RC starts unlit for a strand lym did not launch. Correct per "off until clicked".
- `composeButtonRowLocked`: task place gives an empty `orchDir`. Prime and beam give `AnchorPath`. A render with no entries gives an empty `orchDir`, the same as `@lym_button_dir` before the diff. Correct.
- `pinButtonRowLocked`, task: `orchDirWord = Quote("")` = `''`, so the line is `slack bar --dir ''`. `barSlackWord("")` returns no slack word, so the task draws away alone. Correct.
- `pinButtonRowLocked`, empty render: `orchDirWord` is the bare empty string, which matches how `dirWord` behaved before the diff. Not a regression.
- `composeSlackStateCommand`: now reads `@lym_orch_dir`. Correct.
- `bindingLinesLocked`, RC click line that cannot be composed: the RC option is unset, `remoteControlShownCondition` is false and no button is drawn. Correct.
- `remoteControlShownCondition`, agent pane whose strand has an empty name: `@strand` is set, so the button is drawn, but `@lym_strand_name` is unset, so every click refuses. Wrong against the condition's own contract (finding 2).

**Findings**

1. MEDIUM, design, `internal/reedengine/bar.go` (`remoteControlShownCondition`) with `internal/reedengine/panebin.go` (`composeRemoteControlClickCommand`). The RC button is now drawn in any agent pane, but its click silently does nothing in a session whose `@lym_button_dir` is empty. That happens under a standalone geometry, a failed button context, an unresolvable shell, a failed beam lookup, or a place group with no entries.
   - The condition requires only `@strand` and the click line, never `@lym_button_dir`.
   - The click `cd`s to the empty word, so posix goes to `$HOME`, and the orch `PersistentPreRunE` fails before RunE. `ShowToggleOutcome` never runs and the line discards output, so the operator sees no refusal.
   - Before the diff the button was not drawn there, because `rc-bar --dir ""` printed nothing.
   - Evidence: the bar.go diff around line 591; panebin.go:290 (`ChangeDirToWord("#{"+buttonDirOption+"}")`); `internal/orchcli/cli.go:163-187` (`fail` → `clihelp.Abort` before RunE); `internal/shell/posix.go:108` (`"cd " + word`); `buttonrow.go`'s `composeButtonRowLocked` early returns, which give an empty spec and therefore an empty `dirWord`.

2. LOW, design, `internal/reedengine/bar.go` (`remoteControlShownCondition`). The button is drawn for a strand pane with an empty name, which no click can serve.
   - `@strand` is always set, but `@lym_strand_name` is unset when the name is empty. The click reads `@lym_strand_name` and refuses with "the active pane runs no agent".
   - This contradicts the condition's comment that "a button that no click could serve is never drawn". Keying the condition on `@lym_strand_name` would hold it.
   - Evidence: `internal/reedengine/paneoptions.go:36-45`; `internal/reedengine/remotecontrol.go:37,975`; `internal/orchcli/remotecontrol.go:106-107`.

**Tool calls**
- Read `internal/orchcli/cli.go:163-190`, `internal/reedengine/panebin.go:270-292` and grepped `withOpLock`. My context lacked the orch `PersistentPreRunE` failure path, the full RC click-line composer, and where the op lock is defined, all needed to trace a click with an empty run directory and to check `MarkRemoteControlOn` for a nested lock.
