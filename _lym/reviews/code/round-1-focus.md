---
round: 1
focus:
  - "internal/output/summary.go against the board entry's Idea: does every stop print its error and way forward one line each, at any depth, and does a long plain list collapse to a count with a pointer to the full list? Settle whether leaving object lists uncollapsed, and the five-item threshold for plain lists, satisfy the entry's collapse bullet for a divergence's commit hashes and every run's record."
  - "internal/clihelp/readable.go `UseReadableOutput` and `cmd/lym/main.go` pre-run: does `cmd.Context()` carry the exit state in the pre-run so `onFinish` registers, and does `cmd.SetOut` reach every writer an operator verb uses? Settle which output paths bypass the swapped writer (direct `os.Stdout`, a writer captured before the pre-run) and whether that leaves an operator verb printing the long envelope on a terminal."
  - "internal/output/summarywriter.go `Write` and `Flush`: check the returned byte counts on a partial write error, a held line split across writes, a `{` line that is not an envelope, and a line that never ends. Settle whether any byte can be lost, duplicated or reordered, and whether `Flush` runs on every exit path, including `RunRootCtx` when cobra returns an error."
  - "internal/clihelp/exec.go `RunRootCtx` and `onFinish`: does running the finish steps before the cobra error envelope change the order or content of output for any existing caller, including a nil exit state?"
  - "The `--envelope` flag against existing flags: does the global flag collide with, or change the meaning of, a command's own `--envelope` (`lym creel`) or `--json`, and does it appear correctly in the JSON help and the command index?"
  - "The repository's own rules: does the diff land every doc its constraints document names for a change to CLI output or a global flag, and do the new doc comments follow the repository's comment conventions?"
  - "Summary claims in `_lym/landing/summary.md` against the diff: the label and one-line field lists, the nesting depth of two, the five-item collapse, the `full envelope: rerun with --envelope` trailer, and the four conditions under which the summary prints."
  - "Test economy in `internal/output/summary_test.go`, `summarywriter_test.go` and `internal/clihelp/readable_test.go`: does any test duplicate another's coverage, and does any behavior the diff adds, such as the flush on a cobra error or the `--envelope` shadowing by `lym creel`, lack coverage?"
---

Seed focus for round 1 of a decision-record-only review.
The record holds only the board entry under `## Goal`, so the entry's Idea bullets are the measuring stick and the record's other sections are empty.
