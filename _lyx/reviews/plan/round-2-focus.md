---
round: 2
focus:
  - "Check whether every card that adds a method to `internal/gitrepo`'s `Repo` that calls `r.run` or `r.runChecked` also targets `cmd/lyx/gitrepoboundary_test.go`, whose untagged test asserts set-equality with the literal `gitrepoPinnedRunBoundMethods` list; settle whether the plan stays green under `go test ./...` at each such card's commit."
  - "Check whether the plan's tests reach `PushAnchored` itself, not only `PushPairAnchored`, for the absorbing lock with bounded wait, the `ErrPushLockBusy` expiry, and the fetch-checked retry; settle whether any of the three is left without a test, judged against `PATTERN-test-economy` so that a row added to an existing test counts as coverage."
---

Both entries target the current plan's cards and the plan's own verify section.
