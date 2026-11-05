```yaml
lens: fix-affected-set
outcome: reported
```

Task: check fix-affected-set for F2–F6. The set-shaped findings are F2 (Main's two wiring sites) and F4 (both watched directories).

No findings.

- F2: the only callers of the snapshot and check are `runCheckingGitEntries` (`tmuxkit.go:92`, `:103`) and the tests. No other site in `Main` or elsewhere repeats the sequence. The other `tmuxkit.Main` users are test mains that go through `Main`, so they are covered.
- F4: the table now seeds `dirs[0]` and `dirs[1]`. `runCheckingGitEntries` also uses `dirs[0]`.
- F3, F5, F6: single-site findings with no set to check.
- I left nothing unchecked.

Tool calls:
1. grep for `checkNewGitEntries`, `presentGitEntries` and `runCheckingGitEntries`: my context did not hold the other callers.
2. grep for `tmuxkit.Main` and `gitEntriesBefore`: to confirm no other user kept the old snapshot sequence.
