# Card 2 — readers reject an unquoted parenthesis and a dollar expansion

**Edit:**
- `internal/fabricengine#readOnlySegment`
- `internal/fabricengine#IsReadOnlyCommand`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestIsReadOnlyCommand`

**Create:**
- `plan:internal/fabricengine#readOnlyReaderSegment` -> `func readOnlyReaderSegment(segment string) bool`

**Uses:**
- `internal/fabricengine#blankQuoted`
- `internal/fabricengine#blankQuotedSpans`

**Intent:** Carries D2.
In `internal/fabricengine/refscanner.go`, `readOnlySegment` returns `readOnlyReaderSegment(segment)` in place of `true` when the first field is a listed reader, and its doc comment says a listed reader is read-only only when `readOnlyReaderSegment` admits it.
`readOnlyReaderSegment`, placed beside `readOnlySegment`, reports false when either rule fires and true otherwise:
- Parenthesis rule: `blankQuoted(segment)` holds `(` or `)`.
- Dollar rule: over the rune-aligned pair `view := []rune(blankQuotedSpans(segment, true))` and `raw := []rune(segment)`, some `view[i] == '$'` has a raw follower `raw[i+1]` that is neither whitespace (`unicode.IsSpace`) nor `"`; a `$` at the segment's last rune passes.
  The follower is read from `raw` because the view blanks a single quote itself, so `$'` would otherwise read as a `$` before a space.
Both views keep the segment's length in runes, as `blankQuotedSpans` writes one rune or one space per input rune.
A `$` the view keeps inside double quotes is judged too, and an escaped unquoted `\$` or `\(` is kept by both views and so rejected; both only narrow what is read-only.

`IsReadOnlyCommand`'s doc comment gains, after the sentence naming the readers' forms, that a reader segment is not read-only when it holds an unquoted `(` or `)`, or a `$` its single-quote-blanked view keeps whose next raw character is anything but whitespace, a double quote or the segment's end, which rejects every parameter and arithmetic expansion and the ANSI-C quote `$'…'`, while a regex anchor such as `a$` stays read-only.
Its last sentence, that an expansion and a zsh glob qualifier stay non-read-only, then holds for readers by these rules and is kept.

`internal/fabricengine/doc.go` gains a paragraph right after the `ReferenceRule` paragraph (the two lines ending "shares one rule with the audit."), one sentence per line, leaving the neighbouring fixed-column text as it is.
It states: `IsReadOnlyCommand` (refscanner.go) rejects an unquoted parenthesis and a dollar expansion in a reader segment, because a zsh glob qualifier and a parameter expansion run code with no substitution marker, and otherwise leaves readers' globs and flags free, because no flag of a listed reader writes a file or runs a command, so a glob that expands into a flag-shaped name can do nothing a typed flag could not;
a reader joins `readOnlyCommands` only if none of its flags writes or executes, and a reader that blocks, such as `tail -f`, writes nothing and is admitted;
in `git` words an unquoted glob is rejected except after the `--` of `log`, `show`, `diff` and `status`, because git reads every word after `--` as a pathspec, so an expanded name is a path and never a flag;
every other rule is in `IsReadOnlyCommand`'s comment, which the paragraph points at rather than restating.

Bound: the change only narrows what is read-only; a reader command the new rules reject fails the read-only check and the webster audit treats it as any other non-read-only command.
`lyx`, `git` and `sed` segments are unchanged: their shell-word split already rejects every `$` and unquoted parenthesis.

Test rows join the table of `TestIsReadOnlyCommand` in `internal/fabricengine/refscanner_test.go`;
each reader row reaches `readOnlyReaderSegment` through `IsReadOnlyCommand` after every command-wide scan passes, and the two `git diff` rows reach the git form check:
- false: `cat *(e:'touch x':)`, `ls *(+fn)`, `grep x (a)` (parenthesis rule);
  `cat ${x@P}`, `head -n ${a[y]} f`, `cat $x`, `cat $~x`, `cat $1`, `cat $@`, `cat "${x@P}"` (dollar rule);
  `cat $'\'' $(touch x) f` (dollar rule on `$'`, written as a Go raw string; its `$(` is hidden from the command-wide substitution scan by the unterminated single-quoted span, so only the dollar rule rejects it);
  `git diff *.go` (unquoted glob before `--`).
- true: `cat *`, `grep -rn x --include=*.go internal`, `jq '.a | map(.b)' f.json`, `grep a$ f`, `grep 'a$' f`, `grep "^a$" f`, `git diff -- *.go`.
The existing rows, among them `grep '$(x)' y.txt` true, stay and still pass.

**ImpactSummary:** Narrows `IsReadOnlyCommand` for reader segments only; its callers in `websterengine`, `webstercli` and `loomcli` take it as a predicate, and no existing test feeds them a reader command with a `$` or parenthesis that expects read-only.
