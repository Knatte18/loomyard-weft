# PR-Review row does not notice its session ended

`pr-review-dead-session-unseen` · Task · bug, landing, shuttle · running · PR-Review

Imported from [issue #210](https://github.com/Knatte18/loomyard/issues/210).

Seen in hub mo, run hydraulics-gaps, PR-Review row. The PR-Review session's strand was removed (lym reed remove) at 13:44, and the session logged SessionEnd. The row's lym loom step process went on running with status running at PR-Review, and spawned no new session, until the process was killed by hand at 13:50. Only then did the run mark the row interrupted, and a resume start a fresh session. Expected: the row detects a dead PR-Review session and re-spawns it under the interrupt policy, or halts with a stop the parent can act on.