# Go driver sends no parent message at a watched verify halt

A verify halt that matches the failure record under a watching batten now hands back without a parent message.
Batten sends that run back to its retry target by itself, so the "Run ... stopped; stop report ..." line only asked the orch to answer when it had nothing to do.
Batten's own notice for a second verify failure stays the parent's wake-up.

The hand-back still writes the stop report, commits the run records and parks the run, so the halt stays on record.

## Changes

- `internal/sheddriver`: the rule table's watched-verify row no longer asks for a parent line.
  The `WatchedVerifyLine` field is removed from `driveDecision` and `driveHandBack`, together with its branch in `driveNotices`.
- `internal/loomcli/doc.go`: the Go driver section lists the messages a watched run still sends: the parent notice, a question and a friction-failed done.
- Tests in `internal/sheddriver`: the rows that pinned the watched verify notice now expect none, and the `driveNotices` row for it is gone with the field.

## Design decisions

- No message at all, rather than a line saying no action is needed.
  Any line ends in the driver's answer tail and wakes the orch, which is what this change removes.
- Unchanged: the unwatched verify halt keeps its one goto and its hand-back when the retry is spent, and the other watched notices are still sent.

## Risk

If batten dies after the hand-back, the driver sends the parent nothing for that halt.
Batten's own liveness handling covers that, as for every other watched hand-back that sends nothing.
