# A load turn retries server_error without a cap, and loom status hides the idle time of a stuck step

`load-turn-retry-cap` · Task · bug, shuttle, loom · running · PR-Review

## Seen (issue #102, left out of loop-exit-not-recorded)

A load turn that ends on an API `server_error` makes shuttle wait and retype (`load turn ended on an API error; the prompt waits until the load runs through`), with `apiErrors` rising each time and no cap.
In the reported run it rose 3, 4, 5 before the loop ended with no envelope, and the run history showed hours of idle time around the stuck step that no status line surfaced.
The run `loop-exit-not-recorded` made the loop record its exit status but left these two asks out.

## Idea

- Bound the load-turn retry on `server_error` and name the cap in the halt note, so a halt reads as an API outage rather than only `loop-exited`.
- Show in `lym loom status` the idle time between a stuck step and the next step.