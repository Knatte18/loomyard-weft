---
round: 1
focus:
  - "internal/reedengine/windoworder.go, ownwindow.go, spawn.go: does an overview window open right after the last overview window whose key is not higher than its own, and before the lowest-index window when none? Check equal keys (last opened sits last), add, replace and relaunch alike, and that overview windows stay before every agent, shell and hidden window."
  - "internal/reedengine/strand.go, replace.go, render/types.go: is the order key persisted with the display in reed.json, read only for an overview strand, and does an overview display with no key load as zero? Are the second-overview refusals in AddStrand and ReplaceStrand fully removed, and does UpdateStrand refuse an order-key change naming remove and add?"
  - "internal/reedengine/follow.go, attach.go, bar.go, internal/reedcli/follow.go: do all fallbacks to the overview window take the lowest-index overview window, from reed state's displays for the engine path and from the overview marker for the engine-less follow? Does a live agent window still win, and does a viewer on the lower overview window that ends move to the other overview window, never the shell?"
  - "internal/orchcli/runs.go: does ensureRunsStrand add beside a loom status strand and leave its pane and window untouched, with the runs strand's key a single named constant declared once, keep a current runs strand, and replace a stale one?"
  - "internal/loomcli/bootstrap.go, sharedbootstrap.go, internal/loomengine/driver.go: does statusStrandAddSpec always return an overview display, with the order key derived from the runs strand's constant and the Title the run's recipe name, without reading the session's strands or branching on a recipe name? Is the recipe the one loom start already holds when it ensures the strand?"
  - "Package docs and comments in reed, orchcli, orchengine, loomcli and docs/overview.md: does any statement of one overview strand per session survive, and does reed learn any beam, loom, pawl or recipe name?"
  - "Tests: are the decision record's acceptance criteria each covered at the lowest fitting tier (pure placement and focus functions without tmux where possible), and does any new test duplicate an existing one's coverage? Does any criterion lack coverage, in particular D1 equal keys, D2 viewer-ends, D4 untouched status strand, and D5 no-key load as zero?"
---

Seeded from the decision record's decisions D1 to D5 and the changed-file list of the review range; the diff itself was not read in depth at seed time.
