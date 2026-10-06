# Recovering 10-websterengine-prune over an uncommitted tree

The recovery prompt gave the reason for recovery as `none` and said to continue from the committed tree.
But the worktree held a large uncommitted prune of `internal/websterengine` and its report section from the earlier attempt: 24 files and about 5k changed lines.
Nothing said whether that work was trusted or suspect, so I had to judge it myself.
It built, passed both tiers and matched a fresh redundancy rerun, so I continued on top of it rather than discarding it.
A recovery prompt that names uncommitted changes left by the prior attempt, and says whether to keep them, would remove the guesswork.
