```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checking that each bound, behaviour or fact the fix added or rewrote holds against the code at HEAD.

1. **MEDIUM** — D10's rewritten bound and its criterion: "It runs each engine's `Prepare` into a temporary run directory it removes before it exits, with the parity test's hook runner … Bound: it starts no session, runs no plugin hook and writes nothing outside that temporary directory". The D10 criterion repeats "writes nothing outside a temporary run directory it removes".
   - **Summary:** Codex `Prepare` writes into lym's shared Codex home and runs the real sandbox probe, so the new bound does not hold.
   - **Evidence:**
     - `internal/shuttleengine/codexengine/codexengine.go:114-121`: `Prepare` calls `c.readyHome(codex)`, then `c.launchSandboxMode(codex, home)`.
     - `readyHome` calls `prepareHome` (`codexengine.go:235`). `home.go:140-158` then links the auth file and the sessions directory into `homes.lym`, updates the trust record, and writes the home's `config.toml` through `fsx.AtomicWriteBytes`.
     - `sandbox.go:41,70-77`: unless `sandbox_mode` is full access, `launchSandboxMode` runs `codex sandbox …` as a subprocess.
   - D10 swaps in only the test's hook runner, not D1's faked sandbox probe, and says nothing about the Codex home.
   - The bound needs to name the Codex home's trust record and config as writes outside the run directory, or D10 has to skip or fake `readyHome` and the probe as D1's test does.

2. **LOW** — the new support-log Turn 3 sentence: "a shell string composed through `internal/shell` (`command.go`), so tmux's length limit does not apply".
   - **Summary:** the "so" rests on the launch line being sourced from a script, and neither `command.go` nor the shuttle packages I searched show that.
   - **Evidence:** a grep for a launch script being sourced (`launch.sh`, `source`, `script`) across `internal/shuttle/`, `internal/shuttleengine/*.go` and `codexengine/command.go` found nothing. The claim comes from the reviewer's prose and is not confirmed at HEAD. Name the file that sources the script, or drop the "so".

Claims that held:
- The Claude nudge judges Bash calls with a read-only lookup check: `batchingnudge.go:385-393`.
- The `*` matcher, and one `hooks.PostToolUse` list holding the activity and creel mail entries: `command.go:167-168,247-249`.
- `stencilstore` never overwrites an edited stencil: in `reconcile.go:266-270`, `StateEdited` only raises a notice, and `force` applies only to an untouched stencil.
- `--prompt` and `--prompt-file`: `shuttlecli/run.go:199-200`.
- A block after a tool call reads as a failed call: `codexengine/doc.go:110`.
- `post_tool_use` takes `additionalContext` with `systemMessage`: creel hook-channels report, case 5.
- `codex features list` lists `default_mode_request_user_input` as under development and false.
- The binary holds `project_doc_max_bytes` and `AGENTS.override.md`.
- The `collaborationspawn_agent` payload name: twill spike report, line 127.

Not checkable here: Claude Code's 60-second default hook timeout, its `SessionStart` input and stdout rules, and `CLAUDE_PLUGIN_ROOT` come from external documentation, not this repository. "The lym run's id as `session_id`" is design text and makes no claim about existing code.

Tool calls:
- `strings` on the codex binary plus `codexengine.go:100-140` plus a script grep: the context had no count for `project_doc_max_bytes`, no full Codex `Prepare` body, and no evidence about launch-script delivery.
- A grep of codexengine and claudeengine for writes and subprocesses: to find what `Prepare` touches outside the run directory, which D10's new bound depends on.
- `codexengine.go:215-250`, `sandbox.go:60-100`, and the function list of `home.go`: the bodies of `readyHome` and `launchSandboxMode` were not in the context.
- A script grep plus `home.go:140-165`: to confirm `prepareHome`'s writes and look again for where the launch line is sourced.
