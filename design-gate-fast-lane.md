# A task the operator marks fast lane ranks first in the gate slot queue

`gate-fast-lane` · Note · feature, gate

## Seen

On 2026-11-28 the landing queue task waited behind many other runs for one of two gate slots, while it is the task that would shorten every later landing.
Gate slots rank only by `gateslot.SlotPriority`: the parent lock's holder, then implementers, then the rest.
No task can be put first, so the orch had to pause five unrelated runs by hand to free slots for it.

## Idea

The operator marks a task fast lane on its board entry.
Every gate call of that task's run waits at a rank above `PriorityImplementer`, below `PriorityLockHolder`, so it takes the next free slot before other runs' calls.
The mark is shown on the board and in `lym gate status`, and the operator clears it as easily as setting it.
