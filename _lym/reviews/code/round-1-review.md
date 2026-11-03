---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/ideengine/spawn.go:112
    summary: isBeam degrade to the bare folder, for a missing or unreadable origin record, has no test row although Spawn's doc comment claims it
    fix: In internal/ideengine/spawn_hub_integration_test.go add one row or assertion, in the beam step or beside checkTaskSlugOpensBareFolder, for a pair slug whose origin record is absent or whose weft .weft lock directory is removed; assert Spawn returns no error and CodeLauncher gets the pair's bare anchor folder, not a workspace file.
---

### [LOW] isBeam fallback branches are untested

**Location:** `internal/ideengine/spawn.go:112`
**Issue:** `isBeam` returns false for `found == false` and, with a Warn, for a `ReadOriginFor` error, so the spawn opens the bare folder.
`Spawn`'s doc comment states this degrade as a contract.
`checkBeamOpensBeamWorkspace` covers the marked beam, and `checkTaskSlugOpensBareFolder` covers a task slug with an unmarked record; no row covers an absent or unreadable record.

## Rejected

- Lens code-correctness F-LOW, `seed` launches `BeamWorkspacePath` without checking the regeneration wrote it: `seed` has already resolved the slug through `lymcwd.NamedWorktree` under the hub and written its `.vscode/` config at its anchor, so the layout's skip conditions (outside the hub, missing anchor, prunable) cannot hold; guarding it is code for a state the code's own guarantees rule out.
- Lens code-tests F-T2, no assertion on a beam's window directory: the prime's `windowDir` is equally unasserted before this diff, so it is a standing gap of `placeWindow`'s test surface, not a behavior this diff leaves uncovered.

## Holistic check against the fasit

The Goal's Expected section asks that `lym ide spawn <beam>` regenerate and open the beam's workspace file and a task slug still open its folder.
The diff does both: `seed` regenerates through `fabricengine.RegenerateHubWorkspace`, whose body writes a file for every listed pair whose origin record carries `Beam`, the same mark `isBeam` reads through `ReadOriginFor`, and it launches `BeamWorkspacePath`.
A regeneration error is returned and launches nothing.
`docs/overview.md`, the `spawn.go` header and doc comments and the `idecli/cli.go` header match the code; no caller reads `launchPath` or `windowDir`; no PATTERN entry is touched and the commit carries no departure trailer.
