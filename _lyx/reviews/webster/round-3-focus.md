---
round: 3
focus:
  - "Every tmux set-option or show-option call in internal/reedengine/follow.go, internal/reedengine/lifecycle.go and the tests that write @lyx_follow (internal/reedengine/bindings_integration_test.go, internal/loomcli/smoke_drivingsurface_test.go): does the -t target use a form tmux 3.6 accepts for a session option (=name: or a plain name), and does the argv pinned in follow_test.go match it?"
  - "Engine.Resume in internal/reedengine/lifecycle.go after a server restart: which window does the viewer end on, and does that match the reopened own window with follow on, as TestOwnWindow_RealTmux/ResumeAfterAServerRestartReopensDetached asserts?"
---

Round 3 checks that the two Publish-failure causes are fixed against real tmux, not only through the fake.
