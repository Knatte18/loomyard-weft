# Auto-rebaseline merge-base failure may escape the Stuck mapping

`auto-rebaseline-mergebase-stuck` · Note · bug, webster

## Seen

`webster-replan-state` landed (5a8be0274) with finding `card4-merriambase-failure-escapes-stuck` (MEDIUM) open: an error computing the merge base during auto-rebaseline may surface as a hard failure instead of Stuck.

## Idea

Map that error to Stuck with a way forward, and test it.