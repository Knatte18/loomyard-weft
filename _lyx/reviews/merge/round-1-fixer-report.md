## Changes

- F1: `archivedSlugs` in `tools/tokencount/count.go` strips `refs/tags/` and takes the slug from `fabricengine.ArchiveTagSlug`, replacing its own `refs/tags/archive/` cut and empty-slug/empty-sha checks.
  The empty-commit check on the ref line stays.
  `lyx gate test ./tools/tokencount` is green.

## Disputed

None.

## Deferred

None.
