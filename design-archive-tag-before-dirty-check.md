`internal/fabricengine/remove.go` pushes the archive tag before the no-force dirty checks.
When the sibling is dirty the teardown refuses, but a tag missing the uncommitted records is already on the weft remote, and the resume after `commit-records` pushes another.
Move the archive step after the dirty checks, still before any removal.
Found reviewing PR #304.