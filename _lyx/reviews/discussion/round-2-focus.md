---
round: 2
exclude_lenses: []
focus:
  - "Cancel of a run with no pair worktree (blocked at Worktree-Create): settle whether the decision record's origin-check step and its refusal rules let this case through, given that a run-scope origin reconcile errors when the pair is absent."
  - "`--remove` snapshot and rerun: settle whether the mechanism the record names actually takes the snapshot before force-removal, given that pairteardown snapshots only in local mode, and whether the task branch's fate on origin lets a fresh rerun pass the Worktree-Create leftover probe."
  - "Lock ownership across the cancel sequence: settle whether every lock the record's steps take or reuse (batten's driver, run and status locks, the pair's run lock, the reused cleanup step's own acquisitions) is named by owner, whether any is taken twice in one process, and whether the status lock held across network calls can wait on a hub pause or hibernate."
  - "Declared PATTERN-pair-teardown text against the record: settle whether the record's step order and the point where owned processes end say the same thing as the declared text."
  - "Interruption and offline behavior: settle whether an interrupted `--remove` cancel finishes on a re-run, and whether the unreachable-origin path states what it can no longer judge and what bounds it."
---

Round 1 raised gating findings in every area above, so no lens is excluded.
