---
round: 1
focus:
  - "internal/reedcli/button.go `buttonRunDirectory`: settle whether `lymcwd.ResolveWorktree(dir)` finds the right worktree when `spec.Dir` is the stale rendered directory (the worktree root, an old anchor subdirectory, or a task session's parent or prime anchor), and whether `location.AnchorPath()` then reads the anchor marker afresh rather than a cached or derived value."
  - "internal/reedengine/buttonclick.go: the existence check still runs on `spec.Dir` and the run uses `runDir`. Settle whether `runDir` can be missing on disk or empty when the anchor was changed, and what the click then does; and whether every later use of `spec.Dir` after the anchor read (confirm line, log lines, messages) should be `runDir`."
  - "The confirm path: `composeConfirmedButtonLine` now takes `runDir`, and the confirmed re-entry runs `runButtonVia` again. Settle whether the second pass re-resolves the anchor consistently with the first and whether the prompt's directory change and the later run agree."
  - "Answer key `## Fix` and `## Run instructions`: settle whether the diff confirms the stale-spec cause with a test that sets the anchor marker after the render and then clicks, in `internal/reedcli/button_test.go` and `internal/reedengine/buttonclick_test.go`, and whether the second candidate cause (a task session running in the parent's or prime's anchor, `internal/hubgeom/buttons.go`) is left unaddressed without being a gap."
  - "`RunButton` signature change (new `anchorOf` parameter): settle that every caller and test is updated and that a nil reader cannot reach the click."
  - "Test economy in `buttonclick_test.go`, `buttonclick_integration_test.go` and `button_test.go`: check each new test for overlap with an existing test's coverage and for any behavior the diff adds with no test, such as the unresolvable-anchor message path."
  - "Doc comments and `doc.go` text in `internal/reedcli` and `internal/reedengine`: check the claims (no cwd gate, no git spawn, `buttonRunDirectory` doc, the `cli.go` guard comment edit) against what the code does, and check the repository's constraints document for docs the change must update."
---

Seed focus for round 1. The commit under review is a single one, `ace46fb8`, which has no `Pattern-Departure:` trailer.
The decision record holds the board entry alone, with no plan, so the entry's `## Goal` is the measuring stick.
