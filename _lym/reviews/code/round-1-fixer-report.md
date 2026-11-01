# Fixer-report — round 1, code review

## Changes

One commit, `55087742`, holds all three fixes; they were made together because F1 and F2 share one gate run, not one commit per finding as the fix step asks.

- **F1 (fixed as proposed).**
  `internal/fabriccli/anchordeps.go`: `beamStart` is split into `startDobbiesThroughBeamStart`, which holds the label choice (`root` for the prime, the slug otherwise) and the join of per-worktree failures over an injected run function, and `beamStartChild`, which returns the real `lym beam start` exec call with the `testing.Testing()` guard.
  `EnsureDobbies` calls the two.
  `TestStartDobbiesThroughBeamStart` in `internal/fabriccli/anchor_test.go` pins the prime-to-root mapping, a beam's slug, that one failure does not stop the others, and that the joined error carries each label and the child's output.
- **F2 (fixed as proposed).**
  `internal/orchcli/start.go`: the location choice of `BeamUpWorktree` moved into `bringUpLocation`, which takes the resolver as a parameter.
  `TestBringUpLocation` in `internal/orchcli/startdobbies_test.go` pins prime for the prime's name, the resolved worktree for a beam, and the wrapped resolve error.
- **F3 (fixed as proposed).**
  `internal/orchcli/doc.go` line 11 describes the dobby start as the first step of the pair `BeamUp` brings up, and drops "alone".

Verified with `lym gate test ./internal/fabriccli ./internal/orchcli --tier 0`: both packages pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the callers of `BeamUpWorktree` in batten and pawl are untested, and batten may pass a non-prime location. Disputed, premise false: `internal/battencli/cli.go:106` documents `location` as the resolved location of the hub's prime worktree, and `fabricengine.WorktreePath` is `filepath.Join(l.HubPath, slug)`, so either location resolves the same sibling path.

## Disputed

- fix-proposal finding on the batten wiring, ground `premise false`: evidence is `internal/battencli/cli.go:106` ("location is the resolved *lymcwd.Location for the hub's prime worktree") and `internal/fabricengine/junction.go:26-28`.

## Deferred

Nothing deferred.
