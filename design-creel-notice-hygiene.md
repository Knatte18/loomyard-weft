# Creel sends an unchanged notice once and drops mail about runs that no longer exist

`creel-notice-hygiene` · Task · bug, creel, dobby, fabric, orch · running · Seed-Child

Depends on: [`adopt-handover-test-gaps`](design-adopt-handover-test-gaps.md)

Fix both creel faults in one run. Read first where dobby and fabric decide to send, and why mail waited in the queue.

## From note creel-repeat-notices

## Seen (orch, 2026-11-11)

During a network outage on OSL-1046 (`Could not resolve host: github.com`, `EAI_AGAIN`), the orch's mail held about 25 dobby notices that read the same for `testmap-verb` and `arras-recipe-perch`: same state, same `since` time, same history length.
FABRIC sent the same push failure for the same branch several times, for example the bolt branch `main` and the records branch of each pair.

## Idea

A notice whose text and run state have not changed since the last one sent to the same strand is not sent again, or is sent again only after a stated interval.
Read first where dobby and fabric decide to send, and whether a repeat is deliberate, for instance as a reminder.

## From note creel-stale-mail-delivery

## Seen (orch, 2026-11-11)

The orch received creel mail more than a day old, about pairs the orch had already torn down with `lym cleanup --apply`: `terminal-titles-hub-project`, `testmap-verb` and `arras-recipe-perch`.
It arrived under a tool result when the orch loaded skills, long after the orch's last turn, and the senders' runs no longer existed.
One line asked the orch to settle a stop of a run that was gone.

## Idea

Mail addressed about a run or pair that no longer exists is dropped or marked stale rather than handed to the orch as live.
Read first why the mail waited in the queue, whether the orch's session was down, and whether teardown can clear the strand's pending notices.

