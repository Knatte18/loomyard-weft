---
round: 2
focus:
  - "Comment line breaks: in the Go comments the diff adds or changes in internal/fabricengine (add.go rollbackAdd doc, destroy.go createGitWorktree doc, and any comment a fix touched), does every line break fall at a sentence end, a semicolon, or a comma before a coordinating conjunction whose clause has its own subject and verb, per PATTERN-comment-line-breaks?"
  - "Rename residue: after the warp/weft to code/records rename, does any doc comment, package doc (internal/fabricengine/doc.go), test name or test helper in internal/fabricengine still spell an old name or old side value where the code now uses code or records (for example the PushPairAnchored doc and the error text `push <side> side at <path>`)?"
  - "Test-budget row: does the internal/fabricengine row in cmd/lyx/testdata/test-budget.yaml equal the package's top-level test count at HEAD, counted as PATTERN-test-speed counts, excluding TestMain?"
  - "Gate coverage: do the refusals of ownedCreatedBranch (foreign branch name, empty proof) and dirtyPushedByThisCall (empty leaseSHA on remote delete, local delete) in internal/fabricengine/destroy.go have a test reaching them, added as rows of existing tests under PATTERN-test-economy?"
---

The focus lists where to look; the review still covers the whole range.
