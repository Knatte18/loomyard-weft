# Card 2 — classifier-shell-words

**Create:**
- `internal/fabricengine#shellWord`
- `internal/fabricengine#splitShellWords`
- `internal/fabricengine#dropStderrRedirections`
- `internal/fabricengine#readOnlySedSegment`

**Edit:**
- `internal/fabricengine#IsReadOnlyCommand`
- `internal/fabricengine#readOnlySegment` -> `func readOnlySegment(segment string) bool`
- `internal/fabricengine#readOnlyLyxSegment` -> `func readOnlyLyxSegment(words []shellWord) bool`
- `internal/fabricengine#readOnlyGitSegment` -> `func readOnlyGitSegment(words []shellWord) bool`
- `internal/fabricengine#gitForm`
- `internal/fabricengine#gitForm.admits` -> `func (f gitForm) admits(args []shellWord) bool`
- `internal/fabricengine#historyForm`
- `internal/fabricengine#gitForms`
- `internal/fabricengine#readOnlyGitRemote` -> `func readOnlyGitRemote(args []shellWord) bool`
- `internal/fabricengine#readOnlyGitBranch` -> `func readOnlyGitBranch(args []shellWord) bool`
- `internal/fabricengine#countPositionals` -> `func countPositionals(words []shellWord) int`
- `internal/fabricengine/refscanner_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Delete:**
- `internal/fabricengine#plainWord`
- `internal/fabricengine#plainWordFails`

**Uses:**
- `internal/fabricengine#blankQuoted`
- `internal/fabricengine#readOnlyCommands`
- `internal/fabricengine#lyxCommandWord`

**Intent:**
Close #514 and #518 under D2: a `lyx`, `git` or `sed` segment is tokenised shell-style and judged on dequoted words, the two stderr redirections `2>&1` and `2>/dev/null` are dropped before the redirection scan, and `sed -n <lines>p <files>` joins the readers.

`shellWord` carries the dequoted text, whether any part of the word was quoted, and whether the word holds an unquoted glob character: `*`, `?`, `[` or `#` anywhere, or `^` at the word's start, since zsh's `EXTENDED_GLOB` reads those two as globs.
`splitShellWords` splits one segment's raw text into words honouring single and double quotes, adjacent quoted and unquoted parts joining into one word, and reports false when any word holds `$` or a backslash, quoted or not, an unquoted brace or parenthesis, or an unterminated quote.
An unquoted leading `~` is text like any other character, so a tilde positional and a tilde `-C` directory are admitted without tracking.

`dropStderrRedirections` runs in `IsReadOnlyCommand` before the `>` and `&` scans and before the segments are split: on the quote-blanked view it finds each whitespace-delimited unquoted token spelled exactly `2>&1` or `2>/dev/null` that is not the first token of its segment, and blanks that span in the original with spaces, so the copies stay rune-aligned.
Every other redirection still fails the `>` scan, the same two spelled with a space or another descriptor included.

`readOnlySegment` takes the segment's raw text, reads the command word in its original spelling as today, admits a listed reader with no word check, and for `lyx`, `git` and `sed` calls `splitShellWords` and the segment rule; a split that fails rejects the segment.
The command word and every subcommand word must be unquoted and plain: the `lyx` command words still match `lyxCommandWord`, and the `git` subcommand, `-C` and its directory word, and `remote`'s and `branch`'s subcommand words carry no quote.
A dequoted word starting with `-` is a flag and must match the form exactly as today, so a quoted writing flag is rejected.
In a `lyx` segment a word holding a glob character, quoted or not, is rejected.
`gitForm` gains a field admitting an unquoted glob in a word after `--`, set on `historyForm` and the `diff` and `status` entries of `gitForms`, since only those forms read every word after `--` as a pathspec;
`admits` rejects an unquoted-glob word before `--` everywhere and after `--` where the field is unset, so `ls-remote` and `remote show` reject a glob there, and a quoted glob is text admitted wherever a positional is.
Every `git` word that `admits` does not judge rejects an unquoted glob too, since the shell would expand it into names before git reads them and a planted name could spell a flag:
`readOnlyGitSegment` rejects one in the `-C` directory and in the `cat-file` and `config` words, and `readOnlyGitBranch` rejects one in a `--list` pattern, which admits a quoted glob as text.
Bound: an unquoted glob reaches git only after `--` in `log`, `show`, `diff` and `status`, where every word is a pathspec and none is read as a flag.
`readOnlySedSegment` admits exactly `-n`, then one script word that is a line number or a range of two line numbers followed by `p`, then zero or more file words none of which starts with `-` or holds an unquoted glob, so a print of a pipe's stdin is a read too.

The `IsReadOnlyCommand` doc comment is rewritten to state the forms and bounds above in place of the plain-word sentence: the classifier assumes bash or zsh with any glob option set and sees static shape only, so a loop, an expansion, a substitution, a zsh glob qualifier, a bash extglob and a command behind `bash -c` stay non-read-only.
`internal/websterengine/doc.go`'s audit section sentence on the classifier gains the quoted, glob, tilde, stderr-redirection and `sed -n` rules in one sentence each, and the "audit: fabric-reference" row of `contracts/specs/refusal-spec.md` names the same forms in its condition column.

Rows of `TestIsReadOnlyCommand`, admitted:
`git log --format='%h %s'`, `git log --format="%h %s" -- '*.go'`, `git log --oneline -- *.go`, `git log --oneline -- x#`, `git -C ~/repo status`, `git branch --list 'feat/*'`, `git log '^main' feat`, `lyx fabric --help 2>&1 | head -30`, `cat x | sed -n 1,5p`, and the #518 compound `sed -n 285,300p f.md | cut -c1-200; grep -n "x" -r contracts 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30`.
Rejected:
`git log x#--output=f`, `lyx fabric --help 2>x`, `lyx fabric --help 2> /dev/null`, `lyx fabric --help 1>&2`, `sed -i 1p f`, `sed -n 1p f -i`, `sed -n 1p *`, `sed -n 1w x f`, `for v in a b; do lyx fabric $v --help 2>&1 | head -5; done`, `git log ^main`, `git log '--output=x'`, `git log "$X"`, `git log '$X'`, `git log 'a\b'`, `git log -- *(.)`, `git log -- @(a|b)`, `git ls-remote -- *`, `git remote show -n -- *`, `git branch --list *`, `git -C * status`, `git cat-file -p *`, `git config --get *`, `lyx fabric diff '*'`.
The existing rows "git log with a glob star" (`git log *`), "git log with a glob question mark", "git log with a glob bracket" and "git log with a brace" keep their outcome under the new rules and already pin `git log *.go`, `git log a?`, `git log [a]` and `git log {a,b}`, so none is added again.
Three existing rows flip under the new rules and are reworded: "descriptor redirection" (`cat x 2>/dev/null`) becomes admitted, "git log with a home path" (`git log ~/x`) becomes admitted, and "git log with a quote" (`git log "a"`) becomes admitted as a quoted positional.
The `TestIsReadOnlyCommand` doc comment names the tokenised forms among what it covers.

**ImpactSummary:** every `lyx` and `git` segment rule changes its word type, the plain-word gate is deleted, and three existing classifier rows flip from rejected to admitted; the only caller outside the package, websterengine's injected classifier, keeps the `IsReadOnlyCommand` signature.
