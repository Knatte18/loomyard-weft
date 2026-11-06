```yaml
lens: fix-proposal
outcome: reported
```

**no findings**

I built these cases against each proposal. None broke it:

- **F2, quote handling.**
  - `sh -c 'docker run -e A={skip_file}'` and the double-quoted form no longer match, because the unquoted branch needs `\s|$` after the placeholder and sees the closing quote. Both stay whole, so `FlagDiffMode` and `CheckRetiredPlaceholders` still name them.
  - `sh -c 'run -e A={skip_file} img'` still migrates correctly, because the lead is a space and the end is a space.
  - Re-matching the matched substring inside `argumentRemainder` is safe. The `^` branch of the lead fails when whitespace comes first, and the `\s+` branch then takes it.
- **F3, continuations.**
  - Two continued arguments in a row, `\⏎  -e A=.. \⏎  -e B=.. \⏎  img`, converge over the repeat loop to `\⏎  \⏎  \⏎  img`. That is valid shell continuation.
  - A continued argument that is last leaves a trailing `\⏎  `, which is harmless in a command string.
  - An argument whose end is a newline keeps lead and newline.
- **F2/F3 tests.** Each new row fails with the old regex. The F3 row would have produced `run \ \⏎ ...` before the fix. The F2 rows would have been rewritten, which the new assertion that no lines are returned catches. The marshalled document is the same for `capture` and `compare`, so the two-lines expectation still holds.
- **F4.** `Checkouts` is read only in `judgesnapshot.go` (lines 140 and 197, plus a seam check in `verify.go`). Nothing before `openJudgeSession` on a fresh session calls it, so the injected error reaches the snapshot step. With the mark opened first, the mark would stay open and the new row would fail.
- **F6.** The on-disk `Scope.Volatile` is written only by the `WriteRecords` closure at `expect.go:125`; the later `WriteRecords` closures at lines 152 and 207 touch other fields. The in-memory copy at line 131 cannot mask removing line 125, because the test reads `f.records().Scope` from disk.
- **F1 and F5.** The new constant sits in the block documented as the engine names loomcli's wiring keys on, and `wiring.go:675` was its only literal in `loomcli`. The F5 refusal strings mirror the neighbouring rows, and the gate ran `configreg` green.

Tool calls made: 1. A grep of `expect.go` and `seal.go` for how each `WriteRecords` closure writes the scope. I needed it to see whether the line-131 copy could mask a missing line-125 write, and my context had only part of `expect.go`.
