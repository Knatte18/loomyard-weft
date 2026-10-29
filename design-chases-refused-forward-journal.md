# TestScheduler_Chases refused-forward subtest races alpha's delivery worker

`chases-refused-forward-journal` · Task · bug, creel · running · Code-Burler-Review

## Seen

`TestScheduler_Chases/AChaseTheHolderStillHoldsUnreadIsRefusedForwardingAndStaysWithItsHolder` in `internal/creelengine` fails about two runs in three on idle hardware, since `ba2a346b` (creel stage 3):

- `deliver_test.go:2627: journal entries after the refused forward = 20; want 19`
- `deliver_test.go:2634: alpha's unread senders = []; want the chase still with its holder`

## Cause

The production code is right: the refused `Forward` writes nothing, since its refusal in `verbs.go` comes before any `writer.append`.
`chaseTick` advances the fake clock, the chaser posts the chase through `deliverMailLocked` and `wakeLocked(alpha)`, and alpha's delivery worker, a real goroutine, types the chase and calls `store.markNoticeRead`.
That appends a `read` journal entry and flips the chase to `StatusRead`.
The test waits only for the chaser to re-arm its timer, never for the worker, so the worker's entry lands between the `before` read and the check, or empties the inbox first.
The neighbouring chase tests wait with `awaitChaseLines` before they assert.

## Fix

Hold alpha before the chase tick, as the test near `deliver_test.go:808` does, so the worker skips the notice and the chase stays unread, and add `awaitChaseLines` where the test needs the worker settled.
First check that the chase clock still reaches its first chase under a hold from another sender: `ageChases` exempts only a hold by the message's own sender.
Moving the refusal onto an already-read chase would stop covering the unread case, so it is the fallback.
