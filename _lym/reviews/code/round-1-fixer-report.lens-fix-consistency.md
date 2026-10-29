```yaml
lens: fix-consistency
outcome: reported
```

Findings:

- LOW, `internal/burlerengine/cluster.go:358` (the `splitShellSegments` doc comment): it says "a file-descriptor number attached before it" is dropped.
  After F2 only an unquoted, unescaped digit run is dropped, so a quoted or escaped digit stays an operand.
  The comment states the old, broader rule.
  The suggested wording is "an unquoted file-descriptor number attached before it".
  `doc.go:304` stays accurate: it only says a redirection with its target is no word.

- The rest agrees:
  - The new `wordQuoted` flag is set in the quote-open and backslash-escape cases and cleared in `endWord`.
    The fd branch only resets a word when `wordQuoted` is false, so the flag is already false there.
  - The new test rows match the `fix` keys of F1, F2 and F3, and sit beside the sibling redirect rows.
  - No other twin of the changed code turned up in the two files I grepped.

Tool calls:
1. grep for "file-descriptor" and "redirect" in `doc.go` and `cluster.go`, because my context did not show whether `doc.go` repeats the fd rule.
