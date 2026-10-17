# Go driver sends no parent message at a watched verify halt

At a verify halt that matches the failure record under a watching batten, the Go driver hands the run back so batten can send it to the record's target.
Until now it also messaged the orch "Run ... stopped; stop report ...", ending in the tail that asks for an answer.
The orch had nothing to do, because batten retries the halt by itself.

The driver now sends no message at that hand-back.
It still writes the stop report, records the hand-back settle record and parks the run.
Batten's own notice at a second failure stays the parent's wake-up.
An unwatched verify halt is unchanged: one goto to the record's target, then a hand-back with the run-id line.

## What changes

- `internal/sheddriver`: `driveDecision.WatchedVerifyLine` and `driveHandBack.WatchedVerifyLine` are removed, and `driveNotices` sends a watched run only the relayed parent notice, a question and a friction-failed done.
- `internal/loomcli/doc.go`: the Go driver section's list of the messages a watched run gets drops the watched verify halt.
- Tests: the rule-table row, the notice-set row and the drive-loop row for the watched verify halt now pin that no notice is sent.

## Design decisions

- No message at all, rather than a reworded line saying batten retries and no action is needed.
  Every driver notice ends in the tail asking the orch to answer, so any line would still wake the orch for nothing.
