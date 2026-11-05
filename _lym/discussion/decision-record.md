## Goal

```markdown
Imported from [issue #67](https://github.com/Knatte18/loomyard/issues/67).

Seen: `lym fabric clone --reset --into ~/Code/Knatte18 --shortname scribe https://github.com/Knatte18/scribe-weft https://github.com/Knatte18/scribe` on an existing hub (old layout with _board) failed with 'this repo's recorded shortname is "scr", not "scribe"; refusing to change it', but its envelope reported mutation path_removed '.' recursive and partial: true: the whole existing hub directory was already deleted when the refusal came.

Workaround: rerun `lym fabric clone` without --reset and without --shortname; the hub came back from its remotes. Nothing was lost only because every branch was pushed.

Expected: --reset validates every argument against the remotes (shortname, binding) before removing anything, so a refusal leaves the existing hub untouched.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-05): --subpath is checked before --reset removes anything

Decision: No narrowing: every argument check, --subpath included, runs before --reset removes the existing hub. A --subpath that disagrees with the anchor recorded on the weft, or names a directory absent from the warp, is refused while the old hub is still intact; read the warp's tree from a fetch into a staging location, not from the hub being replaced.

Rationale: The entry's whole point is that --reset never destroys a hub and then refuses; a refusal after deletion for any argument is the same data loss.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
