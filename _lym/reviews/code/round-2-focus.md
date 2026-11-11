---
round: 2
focus:
  - "internal/adoptrecord/watch.go, ownerSubject and covers: does an adopt record that lists a beam slug by name cover that beam's own worktree when the beam has no owner recorded, including under a foreign-owned root, and does a slug the record does not list still walk up to the first beam with an owner?"
  - "internal/fabricengine/ownerarchive.go, archiveOwnerTip: is each of the three outcomes (origin holds the tip as ancestor or equal, origin diverged, branch absent on origin) decided correctly and pinned by a test row, and does the archive never move an origin branch?"
  - "internal/beamcli/ownerarchive.go, archiveOwnerTips and ownerArchiveBeforeRefusal: are the pairs archived the worktree's own pair (slug root for the prime) plus one per run, and does the archive skip while an adopt covers the worktree?"
  - "internal/dobbyengine/loop.go and handover.go: while an adopt hold applies, are the task start, tidy, test-map, landing and pass steps all skipped, and does the archive failure log reset after the owner returns or a hold applies?"
---

First-round review found the adopt-coverage gap in `ownerSubject` as the one gating logic error.
The focus entries name the sites to re-judge from the code as it stands.
