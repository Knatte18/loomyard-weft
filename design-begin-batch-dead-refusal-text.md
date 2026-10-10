# begin-batch terminal-dead refusal does not name the retry and recovery_exhausted

`begin-batch-dead-refusal-text` · Note · enhancement, webster

## Seen

`webster-replan-state` landed (5a8be0274) with finding `d12-begin-batch-dead-refusal-stale` (MEDIUM) open: begin-batch's refusal for a terminal-dead batch predates the second recovery.

## Idea

The refusal names the second-recovery route and the `recovery_exhausted` state that 5a8be0274 added.