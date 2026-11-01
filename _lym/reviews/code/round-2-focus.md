---
round: 2
focus:
  - "In internal/fabriccli/anchordeps.go, the anchor move's restart of dobbies through a `lym beam start` child (`beamStart` and its loop over worktrees): settle whether the label and argument choice (root for the prime, the slug otherwise) and the joining of per-worktree failures are covered by a test that runs their logic, given that `beamStart` returns early under `testing.Testing()`."
  - "In internal/orchcli/start.go, `BeamUpWorktree`: settle whether the prime-versus-beam location choice and its hand-off to `BeamUp` are covered by a test that reaches them despite the early `ErrBeamUpUnderTest` return."
---

Targets the coverage of the new restart paths; the rest of the diff is settled.
