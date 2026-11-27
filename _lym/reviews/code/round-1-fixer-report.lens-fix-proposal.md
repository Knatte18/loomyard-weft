```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the review's proposals that the fix carried out (F1, F2, F4, F5 and the proposed parts of F3).

**LOW**: `closedHeredocPattern`'s `const operator` in `internal/shellguard/rows.go`; a JSON tab between `<<` and the delimiter word is no longer recognised, and no comment names that pass.
- **Case:** `cat <<\tEOF\ny z\nEOF\nsed -i s/a/b/ f`, with a real tab in the shell command, which the payload encodes as the two characters `\t`.
- **Why it now passes:** `operator` is ``<<-?[[:space:]]*('|\\"|\\\\)?``. `[[:space:]]*` does not match the encoded tab, and `\\\\` needs two backslashes. No listed word can start there, and the generic word's first character excludes `\`.
- **Before the fix:** ``<<-?`` + `shellLineUnitPattern*` accepted `\\[^"n]`, so `\t` was read and the command denied.
- **Docs:** neither the row comment nor the `doc.go` list of passes mentions it.
- **Possible change:** ``<<-?([[:space:]]|\\t)*``.
- **Evidence status:** established by reading the pattern, not by running it.

I found no breaking case for the other proposals:
- **F1:** a listed operator cannot fall through to the generic alternative by skipping the `-` or the quote, because both are excluded as the word's first character. The `notWordFrom` signature change renders the same `gh` pattern as before.
- **F2:** the three reaches line up with the `shellScopes` indices.
- **F4:** the empty-body row fails if the listed body's `*` becomes `+`. The unlisted `MSG` row fails if the generic body's `*` becomes `+`.
- **F5:** `doc.go` now states the reach once as the exception, and I saw no sentence contradicting it.

I did not time the larger pattern under `grep`.

Tool calls: none.
