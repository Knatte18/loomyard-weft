```yaml
lens: fix-affected-set
outcome: reported
```

No findings. All ten tests added after `feaa74d0` appear exactly once in the summary's restored list.

Two points that are not findings:
- **Count of 64 versus the review's 65:** the review's 65 includes the rewritten `proves` guard line (`-`/`+` pair in `feaa74d0..HEAD`), which is not a restored test. 54 + 10 = 64 is right, and dropped is 198 − 10 = 188.
- **Tier 1 table:** the summary's "After the last commit" figures (1366 passed, 44.2 s) predate the restored tests. The review did not ask for them to be updated, so I did not check them.

Tool calls:
1. One Bash call listed the `+func Test` lines in `git diff feaa74d0 HEAD -- internal/fabricengine` and counted each name in `_lym/landing/summary.md`. My context did not hold the function-level diff or the post-edit summary counts.
