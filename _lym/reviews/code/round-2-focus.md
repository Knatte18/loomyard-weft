---
round: 2
focus:
  - "internal/hubgeom/beammodes.go (readBeamMode, BeamModes): for a beam whose worktree is absent on the rendering machine, does the committed README or project file ever show a value, or an own-versus-inherited marking, that the beam's runs on its owner machine would not use? Settle also whether what is shown instead still meets the board entry and its added decisions, including for beams nested under an absent beam."
  - "The BeamModes doc comment, the BeamModes paragraph in internal/hubgeom/doc.go, the RenderToDisk paragraph in internal/boardengine/doc.go, the legend written by modeHeader in internal/boardengine/beammodes.go, and _lym/landing/summary.md: does each describe what the code does for an absent beam worktree, an unreadable beam and a Beam PR link whose pull request is closed or merged?"
  - "internal/hubgeom/beammodes.go (primeOfBoard): how does it behave on a records repository whose refs are not files, and does it find the same main worktree other hubgeom callers find?"
  - "internal/boardengine/beammodes_test.go and internal/hubgeom/beammodes_integration_test.go: would each test fail if the behaviour it names broke, with expected values taken independently of the code under test? Cover absent beam worktrees, sources inherited from another beam or an unknown slug, project-file mode lines, the open-beam filter and a failing reader."
---
