---
round: 2
focus:
  - "internal/fabricengine/onerepository.go, oneRepositoryAt and forgetLayouts: settle whether a test pins that a repeated layout read for one worktree is answered from the process-wide cache, that fabric's marker commit through commitAnchorMarker (anchormarker.go) drops kept answers, and that a failed read is not kept."
  - "internal/fabricengine/onerepository.go, the store of a layout answer after a read and the clear in forgetLayouts: settle whether a read that overlaps a concurrent clear can leave a pre-commit answer in the map, and whether any new guard for it is correct and covered."
  - "doc comments of forgetLayouts and layoutAnswers in onerepository.go, and the layout paragraph of internal/fabricengine/doc.go: settle whether each claim about which fabric writes clear the cache and how long an answer holds matches the call sites of forgetLayouts and the board marker retirement in retireBoltAnchorMarker."
  - "internal/fabricengine/doc.go and mergestateactive.go, the two-repository merge probe wording: settle whether the text about a weft cherry-pick, revert or rebase matches what fabric's replay in internal/gitrepo/reset.go runs and holds."
---

Focus targets the round-1 areas the fixer changes; the reviewer still reviews the whole range.
