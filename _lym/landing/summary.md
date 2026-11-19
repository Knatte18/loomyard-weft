# Raddle checks refuse a filed slug the board would refuse

A raddle proposal could pass every check and be approved with a part slug the board then refused at Raddle-File,
as `single-repo-records-on-code-removal` (35 characters, over the board's limit of 32) did.
The proposal checks now apply the board's slug rules to every slug a filing makes,
so such a slug fails the Raddle-Write gate, `raddle-propose` and `raddle-approve` before the operator approves.

## Changes

- `internal/raddleparser`: `BoardContext` gains `ValidateSlug`, the board's refusal of a slug, and `Check` reports a refused project or part slug as the new finding `slug-invalid`.
  The finding carries the board's reason and the way forward: pick a slug the board admits, then run `lym loom raddle-propose` again.
  A nil `ValidateSlug` checks nothing, so contexts built without one, such as Raddle-File's in-write re-check, behave as before.
- `internal/boardengine`: `ValidateEntrySlug` exports the length limit and root reservation every new slug passes; `NewTask` now calls it.
- `internal/loomcli`: the raddle board context sets `ValidateSlug` to `ValidateEntrySlug` followed by fabric's pair slug rule with the hub's junction names, the same rules `lym board` applies to a new task or project slug.
- `contracts/specs/raddle-proposal-spec.md` lists `slug-invalid`.

## Design decisions

- The rule reaches `raddleparser` as a function in `BoardContext` rather than an import, because the package's imports are closed to the standard library and `discussionparser`.
  This also keeps `boardengine.MaxSlugLength` the one declaration of the limit.
- The pair slug rule is included with the length rule: `lym board` applies it to every new task or project slug, so leaving it out would let another class of approved slug fail later.
- The re-check (`RecheckBoard`) is unchanged: slugs do not change between the check and the filing, so the check at the gate covers them.

## Tests

- `TestCheck` gains rows for a refused part slug, a refused project slug, and a context with no slug rule.
- `TestCheck_DependsOnNoteDetailNamesTheWayForward` became the table `TestCheck_DetailNamesTheWayForward`, with a row pinning the `slug-invalid` way forward.
- `TestRaddleBoardContext` asserts the forged hub's context admits a 32-character slug and refuses a 33-character slug, `root` and a slug the pair slug rule refuses.
- The new `TestCheck` rows and `TestRaddleBoardContext` fail on the merge base.
  The red-first gate is waived by the recorded decision "No test seam for the red-first overlay":
  its base overlay cannot put back the changed spec file, a limit filed as the board note `red-first-overlay-non-go-files`.

## Merge danger

**Door:** two-way
**Blast radius:** raddle
