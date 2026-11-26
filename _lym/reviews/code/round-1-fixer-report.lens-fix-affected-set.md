```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun F1's set search (tests calling `enterSession` / `stopWatchedSession`) and check that the fix covers every member.

Unchecked findings: none.

**no findings**

F1's set has two test callers:
- `entersession_integration_test.go:55`: now asserts the lock file is absent before the entry and present after the wait.
- `watchdog_integration_test.go:511-515`: still only registers the wait in `t.Cleanup`. This is not a gap. F1's `fix` key asks for the assertion in `TestEnterSession_DrivesTheToldTmux` only, and one test pinning the wait is enough.

The only other caller is production, `watchdog.go:606`, which the fix doesn't change.

Tool calls:
- `grep -rnE "stopWatchedSession\(|enterSession\("` over `internal/`: my context didn't hold a repo-wide list of callers.
