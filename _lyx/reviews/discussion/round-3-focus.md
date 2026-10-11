---
round: 3
focus:
  - "Does the decision record's treatment of `awaitStartup` (the pre-ready probe loop in `internal/shuttleengine`, which calls `reed.Status` and `CapturePane` once a second for up to `startup_timeout_s`) fit `PATTERN-daemon-wakeups`, as that rule and `pattern/PATTERN-daemon-wakeups.md` read: its bounded-wait sentence, its definition of an unbounded wait, and its backoff rule for a wake that captures a pane? Is whichever reading the record adopts also landed in the background file in the same commit?"
  - "Does the dismissal-gap rule bound every way a dismissal can reach the pane? Check a play that delivered some keys and then failed, a play with an empty sequence, and a first key that failed, against the stray-keypress hazard the gap exists to prevent, at the one-second probe cadence."
  - "Does the account of which `abandonStartup` failures end with the pane's last lines match the code? Check the send-path failures in `internal/shuttleengine` (`sendVerified`, `failUnlanded`, `pasteVerified`, the box readings) and the causes that report no screen, and check that the tail trigger is stated as never-ready with no cause."
---

Focus targets the open design questions on the wakeup rule, the dismissal gap and the pane-tail account.
