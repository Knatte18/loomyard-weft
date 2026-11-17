# Card 8 — plugin-session-start-contexts

**Create:**
- `plan:internal/shuttleengine/claudeengine#RunPluginHook` -> `func RunPluginHook(run shuttleengine.HookRun) ([]byte, error)`
- `plan:internal/shuttleengine/claudeengine#setHookProcessGroup` -> `func setHookProcessGroup(cmd *exec.Cmd)`
- `plan:internal/shuttleengine/claudeengine#sessionStartHooks` -> `func sessionStartHooks(pluginID, root string) ([]pluginHook, error)`
- `plan:internal/shuttleengine/claudeengine#pluginHook` -> `type pluginHook struct`
- `plan:internal/shuttleengine/claudeengine#pluginHookContext` -> `func pluginHookContext(stdout []byte) string`
- `plan:internal/shuttleengine/claudeengine#TestUserSetup_PluginSessionStartContexts` -> `func TestUserSetup_PluginSessionStartContexts(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#TestRunPluginHook` -> `func TestRunPluginHook(t *testing.T)`

**Edit:**
- `plan:internal/shuttleengine/claudeengine#UserSetupSource.UserSetup`

**Extends:**
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine/claudeengine#installedPluginRoots`
- `plan:internal/shuttleengine/claudeengine#NewUserSetupSource`
- `plan:internal/shuttleengine/claudeengine#writePluginInstalls`
- `plan:internal/shuttleengine/claudeengine#pluginInstall`
- `plan:internal/shuttleengine#HookRun`
- `plan:internal/shuttleengine#HookRunner`
- `plan:internal/shuttleengine#OmittedHook`

**Intent:**
`UserSetup` adds the plugin session-start contexts (D8): each `SessionStart` command hook of a plugin the session scope turns on, run the way Claude Code runs it, with its output.

`sessionStartHooks` reads `<plugin root>/hooks/hooks.json` and returns the `SessionStart` entries' `type: "command"` hooks whose entry matcher is empty, `*`, or matches `startup`, with each hook's `timeout` in seconds when set.
An absent hooks file gives none.
An unreadable or malformed one is returned as an error, which `UserSetup` turns into an `Omitted` entry named after the file with the cause, never an error of its own.

For each hook, in plugin-id order, `UserSetup` calls `req.RunHook` with:
`Command` the hook's command, `Dir` `req.WorkDir`,
`Env` the process environment plus `CLAUDE_PLUGIN_ROOT` set to the plugin's directory,
`Stdin` Claude Code's `SessionStart` input, `{"session_id": req.SessionID, "transcript_path": "", "cwd": req.WorkDir, "hook_event_name": "SessionStart", "source": "startup"}`,
and `Timeout` the hook's own timeout, else `req.HookTimeout`.
Bound: only the turned-on plugins' command hooks run, each under that timeout.
`pluginHookContext` takes the stdout as Claude Code takes it: a JSON object gives its `hookSpecificOutput.additionalContext`, and any other output is plain-text context, trimmed.
A non-empty context becomes a `PluginContexts` item named `plugin <id> SessionStart hook`.
Its way forward is to turn that plugin off in the session scope: drop its skills from the role's skills in agents.yaml (`lym config agents`) or from the spec,
or, while claude.yaml's `session_scope` is off, turn it on (`lym config claude`), since then every installed plugin is on.
An empty context, and a JSON object with no `additionalContext`, add no item.
A hook that errors or times out is left out and appended to `Omitted` with its plugin, command and cause; `UserSetup` itself logs nothing, and card 9's `Prepare` warns about each omitted hook.
A nil `req.RunHook` runs nothing and appends each hook to `Omitted` with the reason `not run`, so a caller can list the hooks unrun.

`RunPluginHook` is the real runner, exported so `internal/shuttleengines` wires it into the Codex engine (card 9).
It runs `sh -c <command>` in `Dir` with `Env` and `Stdin` under a context bounded by `Timeout`, logs the spawn and its end at Info through `internal/logger`, and waits on it with `cmd.Wait` (`PATTERN-child-processes`).
So that the bound holds when the command's shell forks a child that keeps the output pipes open:
`setHookProcessGroup`, declared in a file built on Unix only, starts the hook in its own process group and sets `cmd.Cancel` to kill the whole group;
its copy in a file built on other systems leaves `cmd` as it is;
and on every system `cmd.WaitDelay` bounds the wait for the pipes once the context ends.
A run is short-lived and waited on in `Prepare`, never detached, so it registers no `runproc` record, which the pattern requires of a run's detached or long-lived processes.
It returns stdout, and an error carrying the exit status and trimmed stderr on a non-zero exit, or the timeout.

Tests, tier0:
- `TestUserSetup_PluginSessionStartContexts` (parallel table over plugin trees `writePluginInstalls` lays out and a fake `RunHook` recording each `HookRun`):
  a plain-text hook gives its text; a JSON hook gives its `additionalContext`;
  a hook printing nothing, and one printing JSON with no `additionalContext`, add no item;
  a hook whose matcher is `resume` is not run;
  an off plugin's hook is not run;
  a malformed `hooks/hooks.json` runs nothing of that plugin and lists the file in `Omitted`;
  a failing hook is left out and listed in `Omitted` with its cause;
  a nil runner lists every hook in `Omitted` as `not run`;
  the recorded run carries the stdin fields, `CLAUDE_PLUGIN_ROOT` and the hook's own timeout, else the request's.
- `TestRunPluginHook` (parallel table): a command printing its stdin and `$CLAUDE_PLUGIN_ROOT` returns both;
  a non-zero exit returns an error naming its stderr;
  `sleep 5; true`, whose shell forks the `sleep`, under a 100ms timeout returns an error well within a second.

`claudeengine/doc.go` describes the session-start run and its bounds.

**ImpactSummary:** extends the card-7 source with one more output and adds one exported runner; nothing outside claudeengine calls it before card 9.
