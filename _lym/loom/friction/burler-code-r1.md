# Burler fix, code round 1: parent-review checkpoints

F1 asked for a checkpoint that sits at the Bouncer's judge call, a seam the review's `location` key did not name.
Reaching it meant reading `bouncer.go`, the recipe wiring, shuttle's timeout defaults and the judge stencil's home before one edit could start, which cost most of the round.

The per-finding commit rule fought the change set: F1, F6, F8, F10 and F14 share files with each other and with F2 and F5, so the commits group findings by file and name each group.
The fix-check rubric and fork results were what found the judge's `run_timeout_min` mismatch and the missing enforcement of parent findings; neither shows in the review's `fix` keys.
