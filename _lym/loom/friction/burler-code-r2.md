# Fixing code review round 2: hand commit and the entry push

The finding assumed a task push over a hand commit is rejected as non-fast-forward.
Fabric's pair push instead resets the branch to the tip the last transition names and pushes again.
Nothing in `landingshed` says so, and its fake `PushTask` hides it, so the review and the earlier fix both reasoned from a plain push.
See the deferred section of the round-2 fixer report.
