# Creel hands the orch mail about runs and pairs that no longer exist

`creel-stale-mail-delivery` · Note · bug, creel, orch

## Seen (orch, 2026-11-11)

The orch received creel mail more than a day old, about pairs the orch had already torn down with `lym cleanup --apply`: `terminal-titles-hub-project`, `testmap-verb` and `arras-recipe-perch`.
It arrived under a tool result when the orch loaded skills, long after the orch's last turn, and the senders' runs no longer existed.
One line asked the orch to settle a stop of a run that was gone.

## Idea

Mail addressed about a run or pair that no longer exists is dropped or marked stale rather than handed to the orch as live.
Read first why the mail waited in the queue, whether the orch's session was down, and whether teardown can clear the strand's pending notices.
