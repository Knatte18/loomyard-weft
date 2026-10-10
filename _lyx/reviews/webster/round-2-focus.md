---
round: 2
focus:
  - "Test support in `cmd/lyx/stencilseed_integration_test.go` and `internal/hubreconcile/ensure_integration_test.go`: do these tests capture logs through `internal/testkit/logcapture`, or does any test package still declare its own log sink or capture helper?"
  - "Tests for `lyx fabric reconcile` in `internal/fabriccli` (`fabric.go` hub-config and warp-binding commit and push paths): does each failure of a hub-wide config commit or push, a warp-binding commit and a present-path push have a test asserting a non-zero exit with `pairs` and the `*_detail` key in the envelope?"
  - "Tests for `PushRecorded` in `internal/fabricengine/bolt.go`: is a dropped seed commit followed by a second failed push covered, and is a rebase conflict with no seed commits ahead covered, asserting error, clean tree and unchanged tip?"
  - "`reconcileHubWide` in `internal/hubreconcile/walk.go`: when an earlier hub-wide commit is unpushed and a rerun has nothing new to commit, does the build stamp stay absent, or does `doc.go` state the actual guarantee?"
  - "Comments in `internal/reedcli/smoke_switchseed_test.go` that describe `stencilstore.ModeFor`: do they match its current three-way signature and the unstamped-binary mode?"
---

Entries restate the sites to check; they cap no severity and pre-state no verdict.
