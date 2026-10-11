---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedengine/socketfile_test.go:127
    summary: TestRemoveStaleSocket/stale_socket_is_removed is racy under parallel tests; a concurrent fork can keep a closed listener's socket answering, so Publish verify failed
---

### [BLOCKING] TestRemoveStaleSocket/stale_socket_is_removed fails Publish verify

**Location:** `internal/reedengine/socketfile_test.go:127` (`TestRemoveStaleSocket`, with its stale row's setup just below and the `listenUnixSocket` helper at line 15).

**Issue:** The Publish log (`.lyx/verify/verify-4.log`) shows `socketfile_test.go:187: path exists = true, want false` for the row `stale socket is removed`, in a full-repo run where the `reedengine` package took 45 s.
Neither the branch's own commits nor the merge-in `916d8ec72` touch `internal/reedengine` (`git diff 916d8ec72^1 916d8ec72 --stat` and the branch diff list no file there), so the cause is not a code change in this range.
The row builds its stale socket by closing a `net.UnixListener` created with `SetUnlinkOnClose(false)`, and `removeStaleSocket` keeps a path that still accepts a `net.Dial`.
The test is `t.Parallel()` while other tests in the same package spawn tmux and other subprocesses.
Go closes the listener's fd in this process, but a `fork` another goroutine runs between `ListenUnix` and `Close` holds a copy of the listening fd until that child reaches `exec`.
In that window the socket still has a listener, so `Dial` succeeds and `removeStaleSocket` correctly keeps the file; the row then reads the production behavior as a defect.
This is my reading of the failure, not a reproduction: `go test ./internal/reedengine -run TestRemoveStaleSocket -count=300` and three whole-package runs all passed here, and the failure needs the loaded, parallel full-repo run.
`removeStaleSocket` itself matches its doc comment and is not at fault.
`TestRemoveSocketFileOnceGone` (line 77) closes a non-unlinking listener the same way in its gone-server row, so it has the same exposure.

**Fix:** Make the stale fixture independent of fd inheritance.
Either create the stale socket file by `bind` with no `listen` (a bound, non-listening Unix socket refuses `Dial` for any fd holder), in a helper split by build tag so Windows still compiles, or run the stale rows in a test that is not parallel and names the reason in its doc comment, since a serial test runs with no other test forking.
Apply the same fixture change to the matching row in `TestRemoveSocketFileOnceGone`.
Re-run `lyx gate test ./internal/reedengine` and the full `lyx gate test ./...` to confirm Publish verify passes.

No other finding: I read the six task commits (`git diff 98ce55225..HEAD`) against cards 1 to 5 and found each card's code, doc comments, `PATTERN-pane-text-injection.md` line, `template.yaml` comment, `test-budget.yaml` bump and test rows as the cards specify.
`lyx gate test` over `./internal/shuttleengine/...`, `./internal/shedadapters/...` and `./cmd/lyx/...` passes.
