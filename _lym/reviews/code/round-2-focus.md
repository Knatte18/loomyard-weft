---
round: 2
focus:
  - "internal/loomcli/sharedbootstrap.go, seedAndCommitBootstrap step 3: does a re-entry after a failed or interrupted first-seed commit still commit the stencil pins record when it is on disk, so the phase machine's precondition scan never meets it untracked?"
  - "internal/loomcli/start.go and runDriverSpawnAndWait: does start warn and post the parent notice only when it put a driver to work, and do the start long help and loomcli/doc.go describe that behavior?"
  - "internal/loomcli/stencilpins.go, start.go, resume.go and doc.go: does any wording promise more than lym stencil diff renders for a pinned stencil?"
  - "internal/loomcli/resume.go, resumeFromRunState: is the bootstrap lock released before the parent post on every driver-spawning or waking success path?"
  - "Tests under internal/loomcli and internal/stencilstore for the stencil pins: does each new or changed test pin behavior no other test covers, with its stated tier?"
---
