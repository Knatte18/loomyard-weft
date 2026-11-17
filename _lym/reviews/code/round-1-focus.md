---
round: 1
focus:
  - "internal/dobbyengine/leftover.go: does the widened teardown condition read the FSM state and skip a pair that another machine owns, a pause or hold covers, or a kept cancel marks, per D7?"
  - "internal/dobbyengine/leftover.go and internal/pairteardown/leftover.go: does the hand-over local-mode removal run for a run another machine owns, also under a hold, and keep the pair of a run this machine re-adopted?"
  - "internal/dobbyengine/orphan.go and internal/fabricengine/orphan.go: does the first-sight file survive a dobby restart without resetting or skipping the orphan grace, and is it dropped on teardown or when the pair stops being an orphan?"
  - "Orphan teardown: are both branch-commit ages and live processes of the slug checked, uncommitted files committed onto the archived tip first, and a pair with a batten record kept out of the orphans even without a board entry?"
  - "internal/battencli/leftoverpairs.go and dobbyreads.go: does the pre-upgrade kept-cancel record get a finished keep cancel intent at the first dobby start through D1's synchronous path, and does lym beam status list kept pairs with age?"
  - "internal/dobbyengine/loop.go and passes.go: is the kept-pairs notice posted once per dobby start, naming `lym fabric remove <slug>`, and is the hub workspace file regenerated after each teardown?"
  - "internal/tendercli and internal/tenderengine/resume.go: does resume spawn no pair push for a transition yet still pay a push a pause deferred, such as an anchor change's?"
  - "The deleted internal/dobbyengine/ended.go and ended_test.go: is each behavior the old ended-run teardown covered still covered by leftover.go and its tests, and does a deleted function leave no caller?"
  - "Test economy and PATTERN compliance: pattern/PATTERN-pair-remote.md and the PATTERN-code-conventions rule that the orphan grace is one named constant; PATTERN-test-only-code for every new function."
---

Seed focus drawn from the decision record's D7 and its acceptance criteria against the diff's file list.
