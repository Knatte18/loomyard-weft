---
round: 1
focus:
  - "internal/ideengine/spawn.go `seed`: does `fabricengine.RegenerateHubWorkspace` really regenerate every beam's workspace file, so that `BeamWorkspacePath(l, slug)` exists when the beam path launches it? Read the function's body, not only its doc comment."
  - "internal/ideengine/spawn.go `seed`: for a beam the hub workspace file is regenerated as a side effect; does the beam path return the right error and launch path when regeneration fails or the beam's file is absent, and is `windowDir` right for the beam file?"
  - "internal/ideengine/spawn.go `isBeam`: is `ReadOriginFor(...).Beam` the mark that regeneration writes a beam's file by, and does a missing record (`found == false`) or a prime/task slug with an unreadable record still reach the bare-folder path as the Spawn doc comment claims?"
  - "Seed (not Spawn) of a beam and of a task: does the changed `seed` alter what `lym ide seed` writes or launches for a task slug or for the prime, beyond what the Goal asks?"
  - "Callers of `ideengine.Spawn` and `Seed` (idecli, reed code button, menu): does any caller depend on the old result for a beam slug, such as `launchPath` or `windowDir` being the bare folder?"
  - "internal/ideengine/spawn_hub_integration_test.go `checkBeamOpensBeamWorkspace`: does it cover the Goal's behaviors without duplicating `checkPrimeWritesHubWorkspace` or `checkTaskSlugOpensBareFolder`, and does it leave state that later scenario steps rely on?"
  - "docs/overview.md and the `spawn.go` / `idecli/cli.go` doc comments: does each claim (beam file contents, regenerate-on-seed, task slug unchanged) match the code as the diff leaves it, and are other docs that describe `lym ide spawn` for a beam left stale?"
---

Seed focus for a decision-record-alone review: the measuring stick is the Goal's `Expected` section.
The commit message and doc comments are claims to check against the diff.
