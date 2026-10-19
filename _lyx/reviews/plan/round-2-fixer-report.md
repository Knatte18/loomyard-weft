# Fixer report — plan, round 2

## Changes

- F1 (fixed as proposed): in `01-hub-workspace-file.md`, `TestRegenerateHubWorkspace` row 7's changed folder list is now arranged by adding a pair with a raw `git worktree add` after a first call has written the file, as row 12 does.
  The Tests list gains a sentence: a row expecting a call to write after the pair set changed changes the pair set with a raw `git worktree add`, never through `hubforge.AddPair` or another verb.
- F2 (fixed as proposed): in `02-verbs-regenerate.md`, row 5 removes the workspace file first and asserts that the failed `Add` recreates it, listing the prime and `Bolt` and not the pair.
- F3 (fixed as proposed): `04-retire-portals.md` adds the Edit target `internal/fabricengine#applyStalePairOwnership` and a comment-rewording bullet saying its doc names launcher and worktree-registration debris only.
- F4 (fixed differently in part): in `05-retired-artifact-sweep.md`, rows 1 to 3 and `TestSweepRetiredHubArtifacts_UnresolvablePrime` plant their links with `fslink.CreateDirLink`, and `internal/fslink#CreateDirLink` is added to Uses.
  The proposal kept `os.Symlink` for row 5.
  Row 5 uses `fslink.CreateDirLink` instead, because its escaping links are directory links, and card 3 builds the same escaping `_launchers/<AnchorRel>` with `fslink.CreateDirLink` (`03-reconcile-launchers-restored.md:49`).
  Rows 5 and 6 now say "directory link" in place of "symlink".
  Only row 4, which needs a link at a file path, keeps `os.Symlink` with a skip on a permission error.
  In `04-retire-portals.md`, the launcher refusal row now names an `fslink.CreateDirLink` link and says "that link" where it said "that symlink".
  `internal/fslink#CreateDirLink` is added to card 4's Uses.

## Self-check

- fix-consistency, LOW: `TestRemove_ReturnsPairNotFoundWhenNothingRemains`'s new row in card 4 did not name the link type for its hand-planted portal link. Fixed as proposed: it now plants the link with `fslink.CreateDirLink`.
- fix-claims, MEDIUM: card 5 said row 5 needs `os.Symlink`, which is false and contradicts card 3. Fixed as proposed; this is the row 5 change recorded under F4 above.
- fix-proposal: no findings.
- fix-affected-set: no findings. F2 was not checked because its premise is a single row, not a set.

## Disputed

None.

## Deferred

None.
