```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code this fix changed, traced by hand against the code in context.

**LOW — `closedHeredocPattern`, `operator` (`<<-?[[:space:]]*(…)?`): a tab between `<<` and the delimiter word is no longer read, and no comment names this pass.**
- Input: a command with a real tab, `cat <<\tEOF\nx y\nEOF\nsed -i s/a/b/ f`, which JSON encodes as the two characters `\t`.
- `[[:space:]]*` does not match those two characters. The quote or escape prefix does not match them either: `\\\\` needs two backslashes.
- A listed word cannot start at `\`, and `notWordFrom` treats `\` as a word breaker, so neither alternative matches and the edit passes.
- The pre-fix pattern `<<-?` + `shellLineUnitPattern*` consumed `\t` through `\\[^"n]` and denied it, so this is a regression.
- The row comment and `doc.go` list the remaining passes without this one. Writing the gap as `([[:space:]]|\\t)*` closes it.

**LOW — new rows in `rows_integration_test.go`: most branches of `heredocBodyLineOtherThan` are unpinned.**
- Every deny row that closes a listed heredoc has a body of `x`, `y z`, tab then `x`, or nothing. These reach only the "first letter differs" branch.
- Dropping the final `?` on a level would stop blank or prefix lines (`E`, `EO`) counting as body. The same goes for breaking the `unit+` or later-letter branches that admit `EOFX` or `Ex`.
- Either mutation would lose real denials, yet the table would still pass. The allow rows still pass because a heredoc that fails to match denies nothing.
- `prose_after_another_listed_word_in_a_listed_heredoc` does not pin `PYX` either. Its closing `PY` has no trailing newline, so nothing is read after it whether or not `PYX` is accepted as a body line.
- One deny row would cover this: `cat <<EOF\n\nE\nEOFX\n\tq\nEOF\nsed -i s/a/b/ f`.

I traced these and found them correct:
- The `notWordFrom` signature change leaves the gh pattern the same: the bracket order and the optional-group condition are unchanged.
- The generic trie never also matches a listed operator, whether bare, quoted, backslash-escaped or `<<-`.
- `reaches` lines up index for index with `shellScopes`.
- The `0-9-` bracket body is well formed.

I did not run any of these traces against grep. I made no tool calls: the context already held everything this lens needed.
