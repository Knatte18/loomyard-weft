---
round: 1
focus:
  - "internal/creelengine/verbs.go, fold.go, journal.go: does topic supersede drop only the same sender's unread mail under the same topic, and survive journal replay and retention without resurrecting or losing mail?"
  - "internal/creelengine/verbs.go drop-run and internal/pairteardown/runmail.go, teardown.go: does teardown drop exactly the removed run's mail, on every teardown path including a hub with no journal, without touching other runs' mail?"
  - "internal/dobbyengine/runnotice.go and internal/orchcli/notify.go: are run notices tagged with a topic and run so a repeat with unchanged state is superseded, while a changed state still reaches the orch?"
  - "internal/fabricengine/failedpush.go and internal/orchcli/pushnotifier.go: is the same push failure for the same branch sent once rather than repeated, and does a different branch or a changed failure still notify?"
  - "internal/creelcli/goclient.go: failure paths of the new client calls, including a daemon that is down or an old daemon that does not know the new protocol fields."
  - "Goal coverage: the board entry asks for both the repeat-notice fix and the stale-mail fix; does the diff make each, and does anything beyond them appear? Check the pattern/PATTERN-pair-remote.md change against the diff."
  - "Tests: the new tests in creelengine, pairteardown, fabricengine and orchcli against the test-economy bullets, including the redundancy report's candidates."
---

Seed focus for round 1, derived from the board entry in the decision record and the diff stat of the review range.
