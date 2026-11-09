```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Behaviour → pinning test**
- **The ask template stores the escaped `\$A` marker before `%%%`.**
  - `TestRunButtonVia`'s `askFor` helper compares the exact argv, `"\$A%%%"` included (`buttonclick_test.go:57`). It is used by the ask rows around `:110-113`.
  - `TestRunButton_RealTmux/AnswerWithEveryMetacharacterReachesTheFileIntact` also pins it end to end, with a real tmux 3.4 (`buttonclick_integration_test.go:148`).
- **A stored marker that kept its backslash (`\$A…`) has each kept backslash before `$[A-Za-z_{]` dropped.**
  - The row `AnsweredCallFromATmuxThatKeepsDollarEscapesDropsOnlyTheKeptBackslashes` (`buttonclick_test.go:118`) covers a letter, `_`, a digit after `_`, and `{`.
  - The same row covers a doubled typed backslash (`\\$HOME` → `\$HOME`), a non-name `$-`, and `\$_z`.
  - The real-tmux row pins the `$HOME` case.
- **A non-name `$` left alone on a keeping tmux.**
  - The same row covers it with `$-`.
- **A stored marker with no backslash (`$A…`) is only trimmed, so the answer's backslashes stay.**
  - `AnsweredCallFromATmuxThatUnescapesEveryDollarKeepsTheAnswersBackslashes` (`buttonclick_test.go:117`) covers it with `$A\$HOME $-`.
  - `AnsweredCallRunsWithTheStoredAnswerTrimmedAndQuoted` covers the plain case.
- **Trimming happens after decoding.**
  - `AnsweredCallRunsWithTheStoredAnswerTrimmedAndQuoted` stores `$A a b\n`.
- **An empty answer behind the marker takes `ask_default`.**
  - `AnsweredCallWithAnEmptyAnswerTakesTheAskDefault` stores `$A\n`.
- **A stored value with no marker at all (`TrimPrefix` no-op).**
  - `AnsweredStrandEntryRunsWithTheCarriedNameNotThePanes` stores `hi` and passes through unchanged.
  - Real values always carry the marker, so this branch is defensive.
- **The `doc.go` marker text.**
  - Static prose, so it is exempt.

**Findings:** none.
- The two new rows each cover a distinct parser behaviour, so neither is redundant.
- A typed `$$HOME` has no row of its own. It adds no behaviour beyond the regex cases already pinned, so I raise nothing.

**Tool calls:** none. My context already held the diff, the full `buttonclick.go`, and the integration test excerpt.
