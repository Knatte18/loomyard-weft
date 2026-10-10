# Card 13 — read-only-classifier-allow-list

**Edit:**
- `internal/fabricengine#IsReadOnlyCommand`
- `internal/fabricengine/refscanner_test.go`

**Create:**
- `plan:internal/fabricengine#plainWord` -> `func plainWord(word string) bool`
- `plan:internal/fabricengine#readOnlyLyxSegment` -> `func readOnlyLyxSegment(words []string) bool`
- `plan:internal/fabricengine#readOnlyGitSegment` -> `func readOnlyGitSegment(words []string) bool`

**Uses:**
- `internal/fabricengine#readOnlyCommands`
- `internal/fabricengine#substitutionMarkers`
- `internal/fabricengine#blankQuotedSpans`

**Intent:** Turn the read-only classifier into an allow-list of plain words that admits the `lyx` help forms, the `lyx fabric` readers and a closed set of `git` reads (D8 and the parent's addition).
`IsReadOnlyCommand` stays the one classifier, closed and static, keeps its quote-aware segment split and now reads `||` as a separator; it keeps every refusal it has: output redirection, command or process substitution, backgrounding, and a first word that is quoted or not a listed command, so `env`, `xargs`, `bash -c` and a variable stay rejected.
Every word of a `lyx` or `git` segment must satisfy `plainWord`: no quote, backslash, `$`, brace or glob character (`*`, `?`, `[`) and no leading `~`; a segment with any other word is rejected.
Each segment must then be a reader from today's list, or pass `readOnlyLyxSegment`, or pass `readOnlyGitSegment`.
`readOnlyLyxSegment` admits `lyx` followed by lowercase command words then `--help` or `-h` as the last word, and `lyx fabric` followed by exactly one of `list`, `pairs`, `status`, `shortname` alone, `diff` with exactly one word not starting with `-`, or `prune` or `cleanup` with no flag.
`readOnlyGitSegment` admits a `git` whose only global option is `-C <dir>`, in one of the record's forms, where every word starting with `-` before a `--` must be one the form lists, matched exactly, and every other word a positional the form admits: `remote get-url` with `--push` or `--all` then one name, `remote -v` alone, `remote show` with `-n` then names; `ls-remote` with `--heads`, `--tags`, `--refs`, `-h` or `-t`; `rev-parse` with `--abbrev-ref`, `--short`, `--verify`, `-q`, `--show-toplevel` or `--git-dir`; `log` and `show` with `--oneline`, `-n` plus digits, `-<digits>`, `--format=…`, `--pretty=…`, `--stat`, `--name-only`, `--name-status`, `--graph`, `--decorate`, `--all`, `-s` or `--no-patch`, then revisions and paths after `--`; `diff` with `--stat`, `--name-only`, `--name-status`, `--cached`, `--staged` or `--no-color`; `status` with `-s`, `--short`, `--porcelain`, `-b` or `--branch`; `branch` with `-a`, `--all`, `-r`, `--remotes`, `-v`, `-vv` or `--show-current` and no positional, or `--list` then patterns; `cat-file` with exactly one of `-t`, `-s`, `-e`, `-p` then one object; `config` with `--get` or `--get-regexp` then one name and at most one pattern.
The doc comment is rewritten: it no longer says `lyx` and `git` are never read-only, and states the bound that the classifier sees static shape only and admits nothing outside these forms.
Bound: a command the classifier rejects keeps its correctness class wherever a caller consults it; the allow-list opens the listed read forms only.
`TestIsReadOnlyCommand` flips its `git status` row and gains the acceptance rows, the accepted ones (`lyx fabric add --help`, `lyx fabric diff 1a2b3c`, `cat _lyx/fabric/origin.json`, `git -C x remote get-url origin`, `git ls-remote --heads`, `git branch -a`, a pipe of readers) and every rejected one the record lists, from `lyx fabric add x` through `git log $X` and `lyx fabric diff`.

**ImpactSummary:** One fabricengine function and three unexported helpers; its one production caller in webstercli passes it by value and is unchanged until card 14.
