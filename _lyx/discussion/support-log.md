# Support log — adopt-owner-write-stale-manifest

## Interview

A darn run holds no interview; the scope comes from the board entry, imported from issue #844.

## Rejected alternatives

- Filter in `renderOwnerWrite` by checking which manifest names exist on disk before the render: it still fails on a rendered but never committed file the render removes, and it fixes one caller of the Bolt commit routes only.
- Rewrite or drop the stale manifest after a pull: the manifest is local and best-effort, and any other stale source of names would reach `git add` the same way.
- A new `gitrepo` method reading the index: more surface than the HEAD tree read `FileAtRevision` already gives, for an index state the board's write routes do not produce.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the filter live? Self-picked: the two Bolt commit routes in `internal/fabricengine/bolt.go`, through one shared helper.
- What counts as tracked? Self-picked: HEAD's tree holds the path; an unborn HEAD tracks nothing.
