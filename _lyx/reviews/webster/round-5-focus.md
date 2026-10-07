---
round: 5
focus:
  - "Comment line breaks: scan every added or changed Go comment line in the diff, test files and `doc.go` files included, and settle whether any line ends without punctuation while its sentence continues on the next line, or holds two independent clauses (a comma plus and/but/so/or followed by its own subject and verb, or a semicolon) on one line; a line that is only the punctuated tail of an untouched legacy sentence is not a finding."
  - "Comment line breaks, fix-introduced text: read every comment line added by the fix commits `1d6e032a7`, `0887623b7` and `7ec3369e5` and settle whether the rewritten sentences themselves keep one sentence per line and one independent clause per line."
  - "Test parallelism: for every untagged (no build tag) `_test.go` function the diff edits by more than one line, including its subtests, settle whether `t.Parallel()` is its first statement or a comment names the process-global state that keeps it serial; the fix commit `ca4670bbe` claims to have covered the ten tests it edited, so settle whether any other edited test remains serial and whether any added `t.Parallel()` makes a test share global state (`t.Setenv`, package variables, logger output, `TMUX`)."
---

The two keys recur because the defects reappear in newly edited text, so the focus asks for a sweep of the whole diff rather than a re-check of named sites.
