```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, tracing each changed branch of `splitShellSegments` (`cluster.go:362-433`) on concrete inputs.

**Branch list (case run → result)**

- `git branch -a 2>/dev/null`: digit word `2` is dropped as the fd, then `/dev/null` as the target. Words are `git branch -a`. Correct.
- `git branch 2>&1 | head`: the fd `2` is dropped, `>` consumes `&`, and the target `1` is dropped. `|` ends the segment. Correct.
- `git branch 2 > x`: `2` is closed by the space before `>`, so `inWord` is false and `2` stays an operand. Only `x` is dropped. Matches the shell, and is mutating. Correct.
- `git branch 2>x`: shell reads an fd redirect, and the code drops `2` and `x`. Correct.
- `git 2>/dev/null commit -m x`: only `/dev/null` leaves the words, so the subcommand is `commit`. Mutating. Correct.
- `>out git commit -m x` and `2>/dev/null git commit`: the target is dropped and `git` becomes `words[0]`, so both are now flagged. They were missed before. Correct, and an improvement.
- `git branch ">"`: the `>` sits inside the quote, so it goes through the `quote != 0` case and stays an operand. Mutating. Correct.
- `git branch \>`: the escape case writes a literal `>`. Correct.
- Redirect as the last token before `;`, `&&` or a newline (`git branch -a >;`, `... > \n`): `endSegment` calls `endWord`, then resets `dropNextWord`, so the flag never leaks into the next segment. Correct. A bare trailing `>` at end of input is bounds-safe (`i+1 < len(runes)` guards each peek).
- `git branch -r > "out file" foo`: the quote opens after `>`, the whole quoted word is dropped, and `foo` stays an operand. Mutating. Correct.
- `git branch -D`-style target: `git branch > -D foo` drops `-D` as the filename and keeps `foo`. That matches shell semantics. Correct.
- `x>y>z` and `>x>y`: the second `>` calls `endWord`, which drops `x` and resets the flag, and the flag is set again. Correct.
- `>2>x`: the target `2` is mistaken for an fd, but both words are dropped either way. Correct.
- `<<EOF`, `<<<"t"`, `<<-EOF`: extra `<` or `>` characters are consumed and the delimiter or here-string word is dropped. A heredoc body still splits into segments and words, as it did before. This is the open risk the record already names. Not a regression.
- `>|` and `&>`: `|` or `&` ends the segment, so the target becomes a stray one-word segment that starts with no `git` and judges nothing. `git &>/dev/null commit` and `git >|x commit` stay unflagged, but they were unflagged before the diff too. Not a regression.
- `<(...)` and `>(...)` process substitution: `(git` is dropped as the target, and the inner command was never judged before either. Not a regression.
- `git branch "2">x` and `git branch \2>x`: the quoted or escaped digit word is treated as an fd, so it is dropped. The shell reads the operand `2` and a redirect. Result: wrong, see F1.

**Findings**

- F1, LOW, design, `internal/burlerengine/cluster.go:412`: the fd check tests only `word.String()`, and quote removal and escape removal have already run. A quoted or escaped all-digit operand directly before `<` or `>`, such as `git branch "2">x`, `git branch '5'>out` or `git tag \1>x`, is dropped as an fd. The guard then sees `git branch` with no operand and passes a command that creates a branch. This breaks D1's bound that only the redirection target leaves the words, and the doc comment's "an unquoted ... file-descriptor number" claim. The fix is to track whether the current word contained any quote or backslash escape, for example a `wordQuoted` flag set in the quote-open and escape cases and cleared where `word.Reset()` runs, and to treat the word as an fd only when that flag is false. A row such as `git branch "2">x` → true would pin it.

No other branch of the diff produced a wrong result, a swallowed error or an inconsistent state.

**Tool calls:** none made; the manifest already held the splitter, its callers and the test table.
