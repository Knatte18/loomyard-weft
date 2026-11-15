---
round: 1
focus:
  - "contracts/stencils/edit/edit-directive.md: does the new text carry both options the board entry names, a documented link and a warning not to build projects with relative references under it, and is the instruction clear to an agent?"
  - "Is the wording factually right about where `.scratch` points (the weft worktree, per fabricengine/doc.go) and about whom the link is for? The directive says 'your task alone'; check it against the doc comment."
  - "The entry offers a real directory or a documented link, and the diff keeps the link. Does the commit message's reason (go test cache hits) match the existing fabricengine/doc.go rationale, and does the directive text still sit well beside 'Work only in your own worktree'?"
  - "Does anything else carry a copy of the edit directive or assert its text (rendered directives, golden files, other stencils or docs naming the scratch rule) that this diff leaves out of step?"
  - "Doc comments in internal/editdirective/doc.go and internal/fabricengine/doc.go: do they follow the repository's own comment conventions, and does the fabricengine note say the same as the directive about the warp anchor versus the weft target?"
---

Diff is three prose-only files; no Go behavior changes, so test economy and mechanical checks have little to bear on.
