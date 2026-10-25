---
round: 1
focus:
  - "internal/battencli/overview.go and overvieworch.go: does the orch scope (own worktree plus every beam whose `hubgeom.ParentOf` chain reaches it; the prime shows all) exclude the prime's orch and sibling beams for a beam window, and terminate on a cyclic or missing parent chain?"
  - "internal/battencli/overview.go: does the frame draw the dobby lines, a blank line, then the orch lines, with a dobby-less worktree drawing the orch section alone, and is no run drawn twice?"
  - "internal/battencli/overviewdobby.go and internal/beamcli/cli.go: do `DobbyRunCells` run lines keep the old run list's stage detail (`paused <minutes>`, the child's numbered stage in Run-Shed, `status unreadable`), and does `lyx beam status` print the same lines as the pane?"
  - "internal/reedengine/spawn.go, windoworder.go and render/types.go: is `Title` read only for an overview strand, falling back to `overview` when empty, and does the colon-and-space window name survive every name-based window lookup in reed?"
  - "internal/orchcli/runs.go and start.go: does the runs strand spec carry `beam: prime` in the prime and `beam: <slug>` in a beam, with the `runs` role unchanged so a relaunch replaces an older strand?"
  - "The PATTERN invariants the record names (`PATTERN-cwd-resolution`, `PATTERN-pair-parent`, `PATTERN-daemon-wakeups`, `PATTERN-test-speed`) against the rewritten overview.go, and the doc landings the scope lists (orchengine, reedengine, beamcli doc.go, battencli cli.go, docs/overview.md, orch role stencil)."
  - "Test economy in overview_test.go, overviewdobby_test.go and start_test.go: do the rewritten tests duplicate each other, and is any added behavior (Title fallback, orch scoping, cyclic parent) left uncovered?"
---

Seed focus drawn from the board entry's design and the record's acceptance criteria.
