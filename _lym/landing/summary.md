# Tell agents never to build or check out under .scratch, .lym or _lym

Fixes #140.

A task agent built an external .NET harness from a checkout under `.scratch`.
`.scratch` is a link into the weft, so MSBuild saw each project under two paths and failed on a missing project reference.
The edit directive invited this: it explained how a build under `.scratch/` resolves relative references and told the agent to name the working tree's files by absolute path.

## Changes

- **Edit directive, both engine copies** (`contracts/stencils/engine/claude/edit-directive.md`, `contracts/stencils/engine/codex/edit-directive.md`):
  the build-resolution sentences are replaced by a ban.
  `.scratch`, `.lym` and `_lym` hold scratch files and records, never a build:
  nothing is built, restored or compiled under them, and no git worktree, clone or checkout is created there.
  Every spawned role's opening stencil carries this directive, on Claude and on Codex alike.
- **Orch role** (`contracts/orchrole/orch-role.md`): its notes carry the same ban,
  since an orch session gets the role text rather than the edit directive.
- **`internal/editdirective`**: the package doc states the ban in place of the old build guidance,
  and `TestDirective_ReadsTheEnginesCopyWithoutBanner` asserts that each engine's copy contains it.

## Design decisions

- `.scratch` stays a link, as the operator decided; only the instructions change.
- The `fabricengine` package doc still records that a build under the link resolves references from the link's target.
  That is a true fact about the link, and the new ban rests on it, so it stays.
