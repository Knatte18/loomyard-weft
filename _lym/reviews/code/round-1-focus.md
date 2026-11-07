---
round: 1
focus:
  - "internal/heddleengine/settings.go: does the fold and merge follow the record (objects merge per key at every depth, a later scalar replaces an earlier one, lists join without repeats, heddle scalar overwrites the file's)?"
  - "internal/heddleengine/settings.go: is removal of a key or list item that heddle wrote last render and no entry sets now correct, with keys heddle never set kept, including after a Claude-written permission and on a missing or malformed file?"
  - "internal/heddleengine/target.go, store.go, override.go: is an entry whose text is no JSON object refused when written, and do markdown targets keep their behavior?"
  - "internal/heddleengine/shipped.json against the removed .claude/settings.json: does it carry everything but mill@millhouse (ly@loomyard on without a marketplace, scribe@scribe on with the scribe marketplace from GitHub Knatte18/scribe, the full deny list, empty attribution)?"
  - "internal/fabricengine (template.yaml, warpexclude.go, config.go): is .claude/settings.local.json in warp_exclude before the heddle seam fires, and does a hub with its own warp_exclude gain the path through the config reconcile?"
  - "Fabric clone and worktree add: does the seam render the settings file on the prime at fabric clone and on an added worktree, as the record asks?"
  - "Record item asking whether targets should declare their paths with fabric reading them from heddle: does the diff settle it or leave a second edit in fabric's config, and is either consistent with the record?"
  - "Test economy: do settings_test.go and warpexclude_test.go duplicate existing coverage or test helpers the public render path already covers, and is any added behavior uncovered?"
  - "Repository rules: pattern/PATTERN-heddle-instructions.md, docs/overview.md and package doc.go changes land with the code; check the commit message against the diff."
---

Seed focus from the Goal entry and the diff stat; the decision record has no other sections.
