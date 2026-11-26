```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal that the fixer carried out as proposed (lock file absent before `enterSession`, present after `stopWatchedSession`).

no findings

I tried these cases against the proposal and none holds:
- **False failure with the fix in place:** `watchPaneOOMKills` calls `reportPaneOOMKills` before it looks at `ctx`, so an early cancel cannot skip the first read. `withOpLock` creates `.lym` and `reed.lock` before any tmux call, and `running.Wait()` returns only after that goroutine does. The 20-run pass agrees.
- **Vacuous pass:** the absence check is a `t.Fatalf` ahead of `enterSession`, so a fixture or setup step that already made the file would fail the test. With the bare `go` in place, the new assertion failed 10 of 10 runs.
- **Second cancel and `Wait` from the retained `t.Cleanup`:** both are harmless on an already stopped session, and the cleanup still covers the earlier `t.Fatalf` path.
- **Coupling to `.lym/reed.lock`:** the literal matches `reedLockFileName` and the state directory. A rename would fail the test loudly, not pass it silently.

Outside my lens, one sentence: the file header at `internal/reedcli/entersession_integration_test.go:1` still says the file pins only the tmux binary, though the test now also pins the wait.

Tool calls: none.
