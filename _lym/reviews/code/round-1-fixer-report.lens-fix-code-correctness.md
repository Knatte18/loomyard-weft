```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the round's change set (commits `4a83092c6` and `fcbccde0d`).

**MEDIUM**: the rewritten `unquotedWordOtherThan` doc comment in `internal/shellguard/rows.go`.

- **Summary:** "that end matches at every character that stops the run" is false for the excepted word itself.
- **Evidence:** take `dbus-send --system /a`. The run is stopped inside `--system`, at one of its letters, because each branch of the built pattern excludes the next letter of `word` or ends the optional tail. `unquotedWordRunEnd()` returns `(["\\;&|()'<>` + backtick + `]|$)` and matches no letter, which is why the command passes. The claim holds only for the characters a word cannot hold, whitespace aside. `doc.go` states it that way: "every character, whitespace aside, that a word cannot hold". The helper doc also contradicts its own preceding sentence, "Only word among the command's unquoted words keeps the run from reaching that end".

No other findings:

- **Word unit:** `unit(excluded)` builds the same string as before the change.
- **Run end:** `unquotedWordRunEnd()` holds the same character set as the old `searchEnd`, in a different order. No `]`, `^` or `-` sits where the order would change the bracket's meaning.
- **Walk-through:** on `dbus-send / x.y; dbus-send --system / a.b`, the run crosses `/` and `x.y`, and the end matches `;`, so the command is denied. That row passed in the tier 1 log.
- **`settingsGoldenPath`:** Go's dependency order initialises it before the `flag.Bool` usage string. Its read and write sites both use it.
- **Tests:** the fix added none.

Further tool calls: none.
