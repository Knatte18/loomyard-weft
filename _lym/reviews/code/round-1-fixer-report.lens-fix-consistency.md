```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (F1–F4) for consistency with its surroundings.

**MEDIUM**
- **Changed text:** the `unquotedWordOtherThan` doc in `internal/shellguard/rows.go`: "that end matches at every character that stops the run, so a command the run cannot read is refused."
- **Summary:** the sentence claims more than the code does, and contradicts the line just above it.
- **Evidence:**
  - The excepted word also stops the run. With `dbus-send --system`, the run can cross at most `--syste`, because the last position is `(unit("m") unit("")*|m rest)`, and a bare `m` at the word end matches neither branch. It therefore stops at `m`.
  - `unquotedWordRunEnd` is `([` + breakers + `]|$)`, which does not match `m`. That non-match is how the exception works.
  - The preceding doc line says "Only word among the command's unquoted words keeps the run from reaching that end."
  - The twin sentence in `doc.go` is accurate: "every character, whitespace aside, that a word cannot hold".
- **Suggested wording:** "that end matches at every character, whitespace aside, that an unquoted word cannot hold".

I found nothing else. The `doc.go` paragraph removal, the matching `_lym/landing/summary.md` edits, the blast radius line, the golden path variable and the regenerated golden agree with each other.

Outside my lens: `scope.argument()` still spells the same character set as a literal beside the new `unquotedWordBreakers` constant. The fix did not make this worse, so I do not count it as a finding.

Tool calls: none.
