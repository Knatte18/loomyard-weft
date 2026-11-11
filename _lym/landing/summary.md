# shuttle await-forks runs from anywhere under the anchor or in the weft

`lym shuttle await-forks` refused to run when a session's harness had moved its cwd into the weft's `_lym/plan`,
failing with "cwd is outside the recorded fabric anchor subtree".
The shuttle pre-run resolved every verb through `lymcwd.Resolve`, whose strict gate admits only the anchored directory itself.
A session cannot control its harness's cwd moves, so the wait degraded into taking each fork's result as it arrived.

## Change

- `internal/lymcwd` gains `ResolveWithin(cwd)`.
  It accepts the anchored directory, any directory below it, and any directory in the worktree's weft sibling.
  A cwd in a weft sibling resolves to the `Location` of the worktree the sibling belongs to,
  so the hub path, worktree name and recorded anchor match what `Resolve` gives from the anchored directory.
  A cwd elsewhere in the worktree is still `ErrCwdOutsideAnchor`, and the error names the anchored directory.
- The shuttle pre-run resolves `await-forks` through `ResolveWithin`; every other shuttle verb keeps the strict gate.
- The `ErrCwdOutsideAnchor` sentinel text now reads "cwd is not at the recorded fabric anchor",
  since the strict gate it also serves refuses any subdirectory.
- The repeated "recorded anchor or `.`" lookup in `lymcwd` is one helper shared by the three resolvers.

## Design decisions

- The relaxed resolver is a separate function rather than a relaxation of `Resolve`.
  The strict gate stays the rule for every verb that acts on the worktree; only a read-only verb opts out of it.
- A weft sibling is recognized by its `-weft` suffix together with a worktree of the trimmed name beside it,
  so a worktree whose own slug ends in `-weft` with no such neighbour resolves as itself.
- A cwd under the anchor's `_lym` link resolves either way:
  where the link resolves as a symlink the walk lands in the weft, and where a Windows junction stays unresolved it lies under the anchor.

## Docs

`docs/overview.md`, `docs/shared-libs/lymcwd.md` and `pattern/PATTERN-cwd-resolution.md` list `ResolveWithin` beside `Resolve` and `ResolveWorktree`.

## Tests

`TestResolveWithin` (tier 1) builds a checkout with a weft sibling recording a `backend` anchor.
It pins resolution from the anchored directory, a directory below it, the weft root, the weft's `_lym/plan` and a path under the anchor's `_lym` link,
and the refusal, naming the anchored directory, from a sibling of the anchor and from the worktree root above it.
