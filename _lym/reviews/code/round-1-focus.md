---
round: 1
focus:
  - "internal/burlerengine/diffpieces.go: does the splitter hold the per-piece byte cap at every level (package, file boundary, hunk start, line end), including a single line over the cap, binary or rename diffs, and an empty diff?"
  - "internal/burlerengine/diffpieces.go: are piece file names deterministic, collision-free across packages with similar paths, and is the write failure path surfaced rather than swallowed?"
  - "internal/shedadapters/burlerreview.go and internal/shedrecipe/entries_burler.go: is the diff read once per spawning call, and what happens when stage_diff is unset, when Env.CodeSubject is absent, or when the patch read fails?"
  - "contracts/stencils/burler/burler-step-1-explore.md, burler-diff-pieces.md and internal/burlerengine/prompt.go: does the explore step name the pieces as the board entry asks, and does it still behave when a round has no diff pieces?"
  - "contracts/recipes/*.yaml: do the tapestry, darn and sampler Code-Burler-Review rows set stage_diff, and does any non-Code review row wrongly stage a diff?"
  - "Scope against the decision record's Idea: does the diff make changes beyond per-package diff files and naming them in the explore step?"
  - "Test economy: check the two template_test.go tests listed as contained against `TestAnswerParentAsk` assertions before filing."
---

Seed focus for round 1, set from the decision record's Goal and the diff's file list.
