```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F3, F4, F5, F6, F7, F8, F9).

No findings. I built a counter-case for each proposal, and none held.

- **F1:** After the fallback the retry runs the fresh spec with the carry-over block, `resuming` is false, and the success path sends the carried-instructions notice. A resumed attempt that fails non-retryably, or on the last attempt, returns the error with both runs set aside. The next start is then fresh, so nothing re-launches the failed session.
- **F3:** The new test seeds a running candidate with a dead own-window strand and an output file that does not exist. That is the shape on which `stopRecorded` marks the stop as interrupted. The gate passed with the new test in place.
- **F7:** `permkit.Chmod` fails the test instead of skipping it. The existing assertions keep running when it is root, which closes the case where the whole test would be skipped under root. A root runner could still write the 0o555 directory, so the failing-write assertion would then fail. That is an environment risk, not a defect in the proposal. Cleanup restores the mode, so `t.TempDir` removal works.
- **F6:** `Run.Stop` and `StopStrand` both leave `Interrupted` false, so the single assertion holds for both forms in the table.
- **F4, F5, F8, F9:** The text and test changes are consistent with their siblings. No other copy of the stencil clause exists.

Tool calls made (1 of 6):
- `grep` of `internal/testkit/permkit/permkit.go`: to see whether `Chmod` skips or fails under root and whether it restores the mode. My context lacked that file.
