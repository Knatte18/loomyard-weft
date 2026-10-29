```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, for the set-premised findings, that the fix covers every member (fix-affected-set lens).

No findings.

- **F2 (the sites that build or clear a word):** the only places that reset `word` or clear `inWord` are `endWord` and the fd branch.
  `endWord` now clears `wordQuoted`.
  The fd branch only resets when `!wordQuoted`, so the flag is already false there.
- **F2 (the sites that start a quoted or escaped word):** the quote-open case and the unquoted backslash-escape case set `wordQuoted`.
  The in-quote cases at lines 390–398 only extend a word that quote-open already flagged.
  The backslash-newline removal and `#` cases build no word.
- **F2 (callers):** `splitShellSegments` has a single caller, `cluster.go:108`, and its signature is unchanged.
  No twin tokenizer exists, since `dropNextWord` appears only in this function.
- **F1 and F3:** the rows are in place, and each pins a distinct behaviour. I did not check by mutation that each row fails without its code.

Unchecked: none.

Tool calls:
1. `grep` for `splitShellSegments` and `dropNextWord` across the repo's Go files. My context did not show callers or twin tokenizers outside `cluster.go`'s 340–470 range.
