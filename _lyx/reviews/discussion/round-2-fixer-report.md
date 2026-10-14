# Fixer report — discussion, round 2

## Changes

- F1, fixed as proposed: Decision 7 now says the lift by identity runs tender's resume sequence for that one entry, giving each non-orch session the entry recorded one `pauserecord.ResumeLine` through the guarded send and leaving the orch to its watcher, as `lyx resume` does.
  Decision 8 says cancel's lift is the same one, resume lines included.
  A new acceptance criterion says every non-orch session the pause interrupted gets its resume line after the wake.
  The premise was checked against `internal/tenderengine/doc.go`, which says a resume both lifts entries and gives each interrupted session one line.
- F2, fixed as proposed: Decision 6's closing Bound sentence now says that after 120 unconfirmed re-reads the daemon lifts its owned entry as on the wake paths (Decision 7) and queues one orch notice naming the failure.
- F3, fixed as proposed: the sleep-deny criterion now names all five verbs, `systemctl --no-wall hibernate`, `/usr/bin/systemctl hibernate` and `sudo systemctl suspend` as denied, and `systemctl status` and `systemctl status hibernate.target` as passing.
- F4, fixed as proposed: two acceptance criteria were added.
  One says an `--after <slug>` whose run halts before Land stays `waiting` and `status` reports the halt as its reason.
  The other says that when the owned entry was resumed and another hub entry was added before the wake, the daemon lifts nothing, leaves the other entry live and ends `done`.

## Disputed

None.

## Deferred

None.
