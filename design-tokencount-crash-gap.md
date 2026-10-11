# Tokencount counts a crashed turn's gap as working time

`tokencount-crash-gap` · Note · enhancement, shed

Seen at run-time-by-role PR-Gate (PR #550): a last assistant line with a null `stop_reason`, or a session that crashes mid-turn, never ends its working interval, so the gap to the next transcript line counts as working time.

Idea: close an interval with no turn end at its last line (or cap it), so a crash shows as idle instead of work.