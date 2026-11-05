```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each function or branch the diff changes, I ran one concrete case, including a failure of each call it makes. I found no defects.

**List (function or branch: case run: verdict)**
- `HookInput`: `[1]`, an empty payload, `{` with bad JSON, `  {"hook_event_name":"Stop","stop_hook_active":true}`, `SessionStart`. Correct. Non-objects and bad JSON are refused, Stop keeps `StopHookActive`, and an unknown event leaves `Event` empty, which `hookRead` refuses.
- `HookOutput`: empty mail gives `{}` on either event. Mail gives `{"decision":"block","reason":...}`, with `<`, `>` and `&` escaped to `\u00xx`, which is valid JSON. Echo-only gives `{}`. Correct. Dropping the echo is a stated design choice, not a bug.
- `RewakeOutput`: returns nil, 0. Correct, and unreachable because `wakeSession` takes the `IdleWaker` branch first.
- `WakeIdle`:
  - Empty payload: `Unmarshal` error, no process started. Correct.
  - Missing `session_id`: refusal, no process started. Correct.
  - `LYM_CODEX_BINARY` unset: falls back to `codexBinary(Config{})`, which is `codex`. Correct.
  - Failing queue: error carries the trimmed output. Correct.
  - Hang: `CommandContext` kills it at 30 s and an error is returned. Correct.
  - Multi-line text: passed as one argv element. Correct.
  - Stdin is nil, so it reads /dev/null.
- `wakeSession`:
  - `IdleWaker` with a failure: `(0, err)`, so the mail stays unread. Correct.
  - `IdleWaker` with success: `(0, nil)`. Correct.
  - Claude path: writes stderr and returns code 2, a write error returns `(0, err)`. Identical to the old inline code, so Claude is unchanged.
  - Empty `out` from a non-waker: a zero-byte write. Correct.
- `hookWait` and `hookWaitExit` with Codex:
  - Wake error: `writeWake` returns the error, `wroteLines` stays false, and the exit is 0 with the mail unread. Correct.
  - Wake success: `wakeCode` 0, `wroteLines` true, exit 0. Correct.
  - Acknowledgement unsent: exit 0. Correct.
- `creelMailHookCommand`:
  - No flag: prints `{}\n` and exits 0 without starting a process. Correct.
  - Empty `$LYM_STRAND_NAME`: `$f$n` is empty, `[ -f dir/ ]` is false, so it prints `{}`. Correct.
  - Flag present and `hook-read` fails: no stdout and a nonzero exit, a non-blocking hook failure. Acceptable.
  - Flag present with a race that leaves no mail: `HookOutput` gives `{}\n`. Correct.
- `creelStopHookCommand`:
  - Flag absent: prints `{}`, appends the record, so one JSON document. Correct.
  - Flag present and mail handed out: the block is printed, it exits 0, and no record is appended. Correct.
  - Flag present with an echo-only claim, so `hook-read` exits 1: the dropped echo prints `{}`, then the record is appended. Correct.
  - Flag present and `hook-read` fails with no stdout: the record is appended and stdout stays empty. Correct.
  - `in=$(cat)` strips trailing newlines, and the record re-adds one. Correct.
- `appendRecord` and `recordingCommand`: output is the same string as before the split, so no change for the non-creel path. Correct.
- `hookSettings`:
  - Zero `creelDelivery`: the old settings. Correct.
  - With creel: Stop becomes the merged entry plus the async waiter, and PostToolUse gets the creel entry after the activity entry. Order is preserved. Correct.
  - `asyncHookEntry`: `async=true,timeout=86400`, with the command TOML-escaped. Correct.
- `creelWaitHookCommand`: `CODEX_HOME='<home>' LYM_CODEX_BINARY='<bin>' lym creel hook-wait codex` is a valid `sh` env-prefix command. Correct.
- `Prepare`: `Board` set gives a non-empty `flagDir` and `CreelHooks` true. `Board` empty gives the zero `creelDelivery` and `CreelHooks` false. Correct.
- `promptText`: empty notice gives the sentence alone as head. A notice gives `notice + " " + sentence`. Passages and the prompt follow after `\n\n`. Correct.
- `OpenLaunch`: unchanged, no hooks. Correct.
- Moved symbols (`CreelFlagDir`, `StrandFlagNameShell`, `CreelTrustSentence`): the string and path bodies are byte-identical to the Claude originals, and `claudeengine` now delegates. Correct, Claude is unchanged.

**Findings:** none.

**Tool calls:** none. The inherited context already held every definition and call site I needed.
