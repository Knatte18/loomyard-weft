no findings

Traced `views/Home.md`, `../outside.md` and an absolute readme through `PathGuard`, `MkdirAll` and `WriteFile`, including a failure of each call.
Both new test rows fail with the defect in place; every caller of `RenderToDisk` still holds.

Further tool calls:
1. Read `internal/fsx/fsx.go` lines 20-60: the context showed only the first 15 lines of `PathGuard`.
