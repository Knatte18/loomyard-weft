```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code correctness of the fix's changes. I made no tool calls and ran nothing; the cases below were traced by hand through the patterns.

**1. MEDIUM — `sessionBusCommandPattern`'s new `dbus-send` branch (`internal/shellguard/rows.go`): it fails open, so some session-bus calls refused before the fix now pass.**

The branch now matches only if every word up to the end of the command parses as `argumentOtherThan("--system")`. Anything it cannot parse means no match, so the command is allowed. The old `dbus-send` + `wordEndPattern` denied all of these.

- **Escaped quote inside a double-quoted argument:** `dbus-send --session --dest=x / x.y "string:say \"hi\""`.
  - The JSON payload holds `\"string:say \\\"hi\\\"\"`.
  - `doubleQuotedOpenPattern` takes `\\` as a pair, so the following `\"` closes the string early.
  - `hi` and the `\\\"` after it parse as a word.
  - The final `\"` then opens a string that never closes, so no `commandEnd` is reached and the command passes.
- **Tab as a separator:** `dbus-send<TAB>--session / x.y`.
  - JSON encodes the tab as `\t`.
  - `space` matches only `[[:space:]]` or a line continuation, and `commandEnd` has `\\n` but not `\\t`, so the command passes.
  - A tab between any later words does the same.

Neither the pattern comment nor the row comment names these as ways past, and no test row covers them.

**2. LOW — `sessionBusCommandPattern` comment: the list of false denies is incomplete.**

The comment says "A --system in quotes or after a redirect is not read". A `--system` after a `$(...)` or backtick argument is not read either. In `dbus-send string:$(date) --system`, the `(` matches `commandEnd`, so the command is denied. The same applies to a backtick.

**No other defects found:**

- **`argumentOtherThan` nesting:** it rejects exactly `--system`, and accepts proper prefixes, longer words and diverging words. The appended `-` is last in its bracket expression, so it is literal.
- **`commandEnd` per scope:** the three values end the scan correctly for the top level and both script quotes.
- **Line continuations:** the `\\\\` escape unit cannot consume a continuation.
- **Hub tmux flag cluster:** the `-uL` and `-2S` rows would fail against the old pattern.
- **`dbus_send_system*` allow rows:** these would also fail against the old pattern. The new deny rows and the spaced-path allow row only pin behaviour.
- **`shellScope.commandEnd`:** all three literals set the new field.

Tool calls: none; the inherited context was enough.
