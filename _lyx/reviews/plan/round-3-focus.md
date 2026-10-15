---
round: 3
focus:
  - "In `_lyx/plan/17-creel-read-sets.md`, check that the Intent's `Request.All` protocol assertion for `TestDaemon_Protocol` in `internal/creelengine/server_test.go` can run as written: every sender it makes (program claim or external claim) must be accepted by `resolveSender` in `internal/creelengine/identity.go` under the row's `asStrand` setting on Linux, and the asserted bodies must fail if `Request.All` were dropped before `Store.Read` in `internal/creelengine/server.go`."
  - "In the same card, check that the `Read_SetsAndInbox` sequence names, at each read step, which messages are unread and read and the exact messages expected, with no label reused after it was read, so that a row passing the new `all` and `from` semantics cannot also pass the old ones."
  - "In the same card, check that every doc comment or `doc.go` sentence describing which handed-out messages are reserved or unread (the `Reading` type comment and `Store.Read` comment in `internal/creelengine/verbs.go`, the read paragraph of `internal/creelengine/doc.go`) is either named for rewording or already consistent with a read that hands out only unread messages, and that every symbol whose comment changes is among the card's `Edit` targets."
---

Focus entries name where to look; the reviewer still judges the whole plan against the rubric.
