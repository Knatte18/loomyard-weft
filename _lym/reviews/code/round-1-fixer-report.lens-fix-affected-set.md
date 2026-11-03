```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1 premise set.** The set is main's code that calls `ConfigDirs`. I ran `git diff 7f0f65b3 f5724733 -G'ConfigDirs'` over main's range. It touches only `internal/hibernatecli/cli.go`, and the only added `ConfigDirs` use is `chooseResumeOnWake(..., fabricengine.ConfigDirs(location))` at `cli.go:101`.
- **Fix coverage.** The fixed sentence in `_lym/landing/summary.md:28` names `fabricengine.ConfigDirs` and `internal/hibernatecli`'s `hibernateconfig.LoadConfig` call. That is the whole set, so nothing is missed. The wider repo has many other `ConfigDirs` callers, but none is in main's range, so the claim "the one beam change main's commits read" holds.
- **Other mentions in the summary.** The file's only `ConfigDirs` or `Records` mention is the fixed sentence at line 28. No sibling sentence repeats the old claim.

Tool calls:
1. `grep -rn "ConfigDirs("` plus a grep of the summary: to list all callers and summary mentions, which my context lacked.
2. `git diff 7f0f65b3 f5724733 -G'ConfigDirs'` and a `-U0` grep: to confirm which `ConfigDirs` uses main's own commits added, which my context lacked.
