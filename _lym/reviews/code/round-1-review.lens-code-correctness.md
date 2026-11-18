```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each changed function and branch by hand against concrete inputs.

## Cases run

**`importIssue`**, changed control flow (`internal/boardengine/intake.go:129-150`):
- Fold with an issue whose body is `" \n"` and has no comments. `foldIssue` writes, then `EmptyBody=true` and `LocalPaths=nil`. `issueContent` sees the same issue, so the note and the flags agree. Correct.
- Into and slug both given. The refusal comes before any write, so nothing is saved. Correct.
- Neither given: the same refusal message as before. Correct.
- `foldIssue` or `importNote` returns an error. A zero result and the error propagate, and no flags are set. Correct.
- Already recorded. The early return at :122-123 carries no flags; the import is a no-op that prints `recorded`, so no flag is expected. Correct.

**`issueBodyEmpty`**: tried `""`, `" \n"` and `"x"`. Correct.

**`issueContent`** (:200-216):
- Blank body with comments: link line, the empty-body note, then the comments. The whitespace body is no longer appended, where before `" \n"` was. Correct.
- Body plus paths: link line, the paths note, then the body. Correct.
- Title holding `"` or a backtick: `%q` gives Go escaping, so `Say "hi"` renders as `"Say \"hi\""` in the markdown note. Cosmetic; see NIT 3.

**`localPathsNamed` and `localPathPattern`** (:153-184):
- `` `.lym/a` ``, `(.lym/a)`, `[x](.lym/a)`, `".lym/a"`, `wt/.lym/a.`, `.lym/a,` and a path at the start of the text: all matched, trailing punctuation trimmed, duplicates dropped. Correct.
- `foo.lym/x` and `_lym/x`: not matched. Correct.
- Two paths separated by one space: both matched, because the stop character is consumed only as the leading delimiter. Correct.
- `**.lym/x**`, `_.lym/x_`, `|.lym/x|`, `key=.lym/x` and `path:.lym/x`: not matched. `*`, `_`, `|`, `=` and `:` are not stop characters, and the optional prefix must end in `/`. Missed paths; see LOW 1.
- `see .lym/x**`: the path comes out as `.lym/x**`, because the trim strips only `.,;:!?`. See LOW 1.
- `C:\wt\.lym\x`: not matched (backslash separators). See LOW 1.
- `https://host/.lym/x`: the whole URL is flagged as a local path. Rare false positive; see LOW 1.
- Prose such as "the `.lym/` directory": flagged as path `.lym/`. Harmless, since the note then names the directory.

**`intakeImport`**, new keys (`internal/boardcli/intake.go:315-323`):
- `EmptyBody` false and `LocalPaths` nil: both keys absent. Correct.
- Both set and `CommentAndClose` fails: the keys are in `short` before the call, so the error envelope carries them, as `doc.go` says. Correct.

**`appendAttachments`** (`internal/selfreportcli/cli.go:196-217`):
- No paths: body returned unchanged. Correct.
- Nil body plus one attachment: the body is the section alone. Correct.
- `--body ""` plus an attachment: the empty body is dropped and the body is the section alone. Correct.
- `--body -` from stdin plus an attachment: stdin is read in `resolveBody`, then the attachment is appended. Correct.
- `--attach -`: `os.ReadFile("-")` fails with `failed to read --attach "-": open -: no such file…`. Stdin is not supported for attachments, but the error is clear and no doc claims it is. Correct.
- Missing file: the error is wrapped with the path, and nothing reaches the transport. Correct.
- Empty file: the section holds an empty fenced block. Correct.
- Invalid UTF-8: each invalid byte counts as one rune, so the limit check stays conservative. The JSON encoding downstream substitutes U+FFFD. No error is swallowed. Correct.
- A total of exactly 65536 runes passes and 65537 is refused. This matches GitHub's limit of 65536 characters. Correct.
- The body without `--attach` is still unchecked against the limit. That is the behaviour at the review base, not a change.

**`attachmentSection`** (:221-233):
- Content `` ```go\nx\n``` ``: longest backtick run 3, so a fence of 4. Correct.
- No backticks: a fence of 3. Correct.
- Content not ending in a newline: the closing fence still sits on its own line. Correct.
- A base name holding a backtick, such as ``a`b.txt``: the heading's inline code span breaks. Cosmetic; see NIT 2.

## Findings

1. **LOW** · design · `internal/boardengine/intake.go:153-159` (`pathStopChars` and `localPathPattern`)
   The detector misses real `.lym/` paths and keeps trailing markup.
   - A path directly after `*`, `_`, `|`, `=` or `:` is not matched: `**.lym/shuttle/x**`, `|.lym/x|`, `key=.lym/x`. Those characters are not in `pathStopChars`, and the optional prefix must end in `/`.
   - A path followed by trailing emphasis keeps it (`.lym/x**`), because :177 trims only `.,;:!?`.
   - Backslash-separated Windows paths are never matched.
   - An `https://…/.lym/…` URL is flagged as a local path.

   Evidence: `internal/boardengine/intake.go:154`, `:158-159` and `:177`.
   Fix: add `*`, `_`, `|`, `=`, `{` and `}` to the stop set (`:` needs care because of Windows drive letters), and accept `\` as a separator.

2. **NIT** · design · `internal/selfreportcli/cli.go:232` (`attachmentSection`)
   A base name holding a backtick breaks the heading's single-backtick code span.
   Evidence: `cli.go:232`, format string ``"## Attached `%s`…"``.
   Fix: wrap the name in a backtick run longer than any run in the name, as is already done for the content fence.

3. **NIT** · design · `internal/boardengine/intake.go:203` (`issueContent`)
   `%q` renders the title with Go string escaping, so a title holding `"`, `\` or a non-printable character shows escaped in the markdown note.
   Evidence: `intake.go:203`.
   Fix: quote the title as markdown, for example `"%s"` or a code span, instead of `%q`.

No other correctness defect: `importIssue`'s result flags and the note in the entry body come from the same issue and cannot disagree.

Tool calls: none.
