```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes on one concrete input.

- **`tmuxScriptCommandPattern` with the incident command** (`rows.go:398`): the JSON payload is `"command":"(script -qfc \"tmux -S s attach -t a\" /dev/null &lt;/dev/zero &gt;/dev/null 2&gt;&amp;1 &amp;)"`. `topLevelReachPattern` takes zero units and then the `(` separator, the prefix matches nothing, `script` matches, and the space after it ends the word. Denied. Correct.
- **Bare `script` at the end of the payload**: `"command":"script"`. The closing JSON `"` sits in `wordEndPattern`'s class. Denied. Correct.
- **Path forms** (boundary):
  - `./script.sh`: whether or not the path prefix takes `./`, the next character `.` is not a word end, so it passes. Correct.
  - `script/bootstrap` after `;`: the empty prefix leaves `/`, which is not a word end, and the prefix `script/` leaves `bootstrap`, so it passes. Correct.
  - `tools/script` is denied, which the row's comment states.
- **Words that only contain the names**: `tmux-mem`, `script-foo` and `scripts` all pass because `-` and letters are not word ends. Correct.
- **Quoted text and arguments**:
  - `git commit -m "fix; tmux attach"`: the double-quoted string is one unit, and the escaped quote cannot open a separator inside it, so it passes. Correct.
  - `grep -rn tmux x`: `tmux` is not in command position, so it passes. Correct.
- **Substitution, pipe and nested shell**:
  - `x=$(tmux ls)`: the `(` separator puts `tmux` in command position. Denied. Correct.
  - `cat f | xargs tmux`: `xargs` is an allowed prefix. Denied. Correct.
  - `bash -c 'tmux list-clients'`: reached at the script's start. Denied. Correct.
- **Commands the prefix list does not cover**: `sudo -u x tmux`, `timeout 5 tmux` and `nohup script …` pass. These are known guardrail gaps; the row's comment names `nohup` and `timeout`, and the `sudo` flag case is the shared `commandWordPrefixPattern` behaviour every row has. No finding.
- **Claude `shellSwitchInstalled` / `shellSwitchAnnounced` for `SwitchAutonomous`** (`settings.go:378`, `settings.go:388`):
  - `buildSettings` gets `spec.Interactive` (`claudeengine.go:269`) and puts it into `denyInputs` at `settings.go:456` and `settings.go:565`.
  - For `interactive=false`, both maps return true, so the row is installed and announced. For `interactive=true`, both return false.
  - The init panic for an unknown switch passes because both maps hold the key. Correct.
- **Codex `switchInstalled` / `switchAnnounced`** (`guards.go:50`, `guards.go:60`): `codexengine.go:180` is the only production `guardInputs` literal, and it sets `autonomous: !spec.Interactive`. That gives the same truth table as Claude.
  - `installedGuards` would panic on an unknown switch, and both maps hold this one. Correct.
  - The `denyNotice` at `codexengine.go:188` gets the same `guardIn`. Correct.
- **Steer and notice texts**: `steerTmuxScriptDeny` holds no `'`, `"`, `\` or line break, so the init checks in shellguard and both engines pass. `openSteersWithWholeCommandRefusal` prefixes the steer, and the notice stays unprefixed, as on the other rows. Correct.
- **Manifest candidates in both engines**: they list every row from `shellguard.Rows` and keep one only when its rendered command is in the written artifact. An interactive launch writes no tmux-script hook, so the row drops from its manifest. Correct.
- **Changed engine tests**: the `wantBareEntries` count of 7 or 8, the new `wantTmuxScript` assertion, the codex `guardIn` with `autonomous: !tt.interactive`, and the `SwitchesOff` row that drops `tmux-script` all match the installed and announced truth tables above. Correct.

Findings: none. Every branch I ran gives the correct result. The changed code makes no call that can fail, so it has no error path to swallow or misreport.

Tool calls:
1. Read `internal/shellguard/rows.go` lines 100–175 (`commandFieldPattern`, the unit patterns, `commandWordPrefixPattern`) and grepped the `buildSettings` callers. My context did not hold those pattern pieces or the site that passes `spec.Interactive`, and I needed both to trace the inputs.
