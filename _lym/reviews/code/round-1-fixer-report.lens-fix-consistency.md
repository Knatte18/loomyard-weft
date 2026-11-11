```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for sibling text left out of step with the changes (lens fix-consistency).

**LOW — `internal/adoptrecord/doc.go:21-22` (package doc, untouched by the fix)**
- The package doc still says "A worktree's subject is the entry whose owner it reads, resolved up the parent chain to `root`. A beam with no owner of its own is therefore covered by the adopt of the subject above it."
- The F1 fix changed the rule. `ownerSubject` now stops at a slug the record lists (`listed(slug)`). `Watch.Covers` (`internal/adoptrecord/watch.go:32-36`) was updated to say so, but the package doc was not.
- The sentence now omits the case F1 added, a beam listed in the record with no owner of its own. Anyone reading the package doc would still expect the old walk to the parent.
- Suggested change: add to doc.go line 22 that a beam the record lists is its own subject.

Nothing else found out of step:
- `ownerSubject` has one caller, and it was updated.
- The `archiveFailures` and `archiveFailing` comment at `internal/dobbyengine/loop.go:320` still holds.
- The handover.go file header and the `Deps.UnderAdopt` docs make no claim the F5 change breaks.

Further tool calls (2):
1. grep for `ownerSubject(`, `archiveFailing` and `no owner of its own` outside tests, to find sibling text and callers of the changed code. My context didn't hold other docs.
2. Read `internal/adoptrecord/doc.go` lines 14-27, to see the package doc's wording of the cover rule. My context didn't hold that file.
