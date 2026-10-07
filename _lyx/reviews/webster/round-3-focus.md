---
round: 3
focus:
  - "Derive the set of `//` comment lines the review range adds with `git diff -U0 $(git merge-base main HEAD)..HEAD -- '*.go'`, then check each added or rewritten sentence against PATTERN-comment-line-breaks: does every such sentence sit on its own line, with no new sentence sharing a line with the tail of an old one and no one-phrase fragment line left behind? Untouched wrapped sentences with a pure word swap or deletion are exempt."
  - "Check `internal/shuttlecli/send.go` and its tests in `internal/shuttlecli`: is there a test running the `send` verb over `shuttlefake.Reed` and `shuttlefake.Engine` that asserts the typed text ends with `shuttleengine.MessageTail`, and that text already ending with it is typed once?"
  - "Check `finalize` in `internal/shuttleengine/wait.go` and the `gateLastDone` memo in `internal/shuttleengine/run.go`: does any call still pass a non-empty message, and does any test compare two values that can only be empty?"
  - "Run `gofmt -l` over the Go files the review range touches and settle whether any is unformatted because of this diff, not the base commit."
---

The four entries are self-contained and name the sites to check.
