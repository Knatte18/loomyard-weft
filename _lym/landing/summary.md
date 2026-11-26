# Settings golden update mode; document guard row exceptions

## Settings golden update mode

A shell guard row change rewrites the Claude settings golden, and until now each such change needed a temporary test file to render it.
`TestBuildSettings_MatchesGolden` now takes an `-update` flag,
under which it writes `internal/shuttleengine/claudeengine/testdata/settings-golden.json` instead of comparing against it:

```
lym gate test ./internal/shuttleengine/claudeengine -- -run TestBuildSettings_MatchesGolden -update
```

Without the flag the test checks the same things as before:
a missing label, a differing settings document or notice, and a label in the golden that the test does not render.

Closes #257

## Exceptions in shell guard rows

A row that refuses a command unless it carries a flag now has a documented way to say so.
Rows stay AND-only.
An exception is spelled inside the pattern that matches the command:
`unquotedWordOtherThan` scans that command's unquoted words, and the new `unquotedWordRunEnd` ends the scan.
That end matches the payload's end and every character, whitespace aside, that an unquoted word cannot hold,
so a command the scan cannot read is refused.
The word pattern and the end are built from one character set, `unquotedWordBreakers`.

The package doc of `internal/shellguard`, the `Row.Patterns` field comment and the `unquotedWordOtherThan` doc state this,
and the package doc points to the session-bus row's dbus-send form as the example.

The session-bus pattern now lists the end characters in a new order with the same meaning,
so the golden changes with it.
A new case in `TestRows_PatternsDenyAndAllow` pins that in `dbus-send / x.y; dbus-send --system / a.b`
the first command is still denied.

Closes #256

## Design decisions

- AND-only rows plus a documented helper, rather than a field of patterns that must not match.
  Such a pattern would match anywhere in the payload,
  so a second command carrying the flag would let a refused first command through.
- `-update` is defined only in the claudeengine test package, so the `lym gate test` call names that one package.

## To check by hand

- The fail-closed guidance for row exceptions, on `unquotedWordOtherThan` and `unquotedWordRunEnd` in `internal/shellguard/rows.go`
  and in `internal/shellguard/doc.go`.
  The finding was that the docs promised a fail-closed scan while the helper returns only the word pattern:
  the end that makes it fail closed was built separately by the caller and described too narrowly,
  so a row written from those words could fail open.
  The diff now builds the word pattern and the end from `unquotedWordBreakers` and names `unquotedWordRunEnd` in the docs;
  check that this settles it.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
