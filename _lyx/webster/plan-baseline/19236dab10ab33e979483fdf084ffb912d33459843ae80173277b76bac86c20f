# Card 6 — sed-print-classifier

**Edit:**
- `internal/fabricengine#sedPrintScript`
- `internal/fabricengine#readOnlySedSegment`
- `internal/fabricengine#IsReadOnlyCommand`
- `internal/fabricengine_test#TestIsReadOnlyCommand`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D5: the fabric-reference read-only classifier admits the `sed` print forms agents read code with, such as `sed -n '/^func X/,/^}/p' f`, so a harmless fabric reference is no longer a correctness halt.

The `sed` form stays `sed`, then the quiet flag, then one script word, then file words, and widens to:
- the quiet flag as `-n`, `--quiet` or `--silent`, in `readOnlySedSegment`;
- a script of one or more print commands joined by `;`, each an address or an address range followed by `p`, with nothing after the last `p`, in `sedPrintScript`;
- an address that is a line number or a regex `/RE/` whose `RE` is non-empty and holds no `/`, `;` or newline, with no flag after its closing `/`.
The word-splitting rules stay: no `$` or backslash in any word, so `$p` and an escaped `/` stay non-read-only, and the file-word check (no leading `-`, no unquoted glob) is unchanged.
Every other script (`s`, `w`, `r`, `e`, a trailing `;`, an empty regex, a regex flag such as `I`) and `-i`, `-e`, `-s` or `-z` anywhere stay non-read-only.
Bound: the widening admits only reads; the grammar has no command that writes, and a regex address cannot hide one because `/`, `;` and newline end or split nothing inside it.

`IsReadOnlyCommand`'s doc comment states the admitted `sed` form in place of its line-number-only sentence, and `readOnlySedSegment`'s and `sedPrintScript`'s comments follow.
`internal/websterengine/doc.go`'s audit section sentence on `sed` states the same form.
The `audit: fabric-reference` row of `contracts/specs/refusal-spec.md` replaces `sed -n <lines>p` with the quiet flag and print scripts of line-number or regex addresses; the row names no denied form unless "never" precedes it.

`TestIsReadOnlyCommand` gains read-only rows for `sed -n '/^func X/,/^}/p' f`, `sed --quiet 3p f`, `sed --silent 3p f`, `sed -n '1,5p;20,30p' f` and `sed -n '/a/p' f`.
It gains non-read-only rows for `sed -n '1w out' f`, `sed -n 's/a/b/w f' f`, `sed -n '$p' f`, `sed -n '/a;b/p' f`, `sed -n '1p;' f`, `sed -n '//p' f`, `sed -n '/a/Ip' f` and `sed -n 3p -i f`; its existing `sed -i 1p f` and `sed -n 1p f -i` rows already cover the other two rejections.

**ImpactSummary:** Widens one read-only classifier branch that the webster audit and recovery verbs consult; only `sed` print forms newly read as read-only, and every writing form stays rejected.
