# TestScheduler_Chases fails: a refused forward grew the journal

`chases-refused-forward-journal` · Note · bug, creel

## Seen

The adopt tier0 run failed `TestScheduler_Chases/AChaseTheHolderStillHoldsUnreadIsRefusedForwardingAndStaysWithItsHolder` in `internal/creelengine`, with a machine under gate load:

- `deliver_test.go:2627: journal entries after the refused forward = 20; want 19`
- `deliver_test.go:2634: alpha's unread senders = []; want the chase still with its holder`

It was not rerun.
Either the refused `Forward` wrote a journal entry, or the chase scheduler fired between the `before` read and the check.
The latest commit touching `creelengine` is `ba2a346b` (creel stage 3).

## Idea

Run the subtest alone repeatedly to tell a timing race from a regression, then fix the test or the scheduler deterministically.
