---
round: 2
focus:
  - "internal/fabricengine/add.go, Add: settle whether any refusal path (no remote, missing weft repo, ErrRecordsWorktreeNotPair, refuseAdoptLocalCheckedOut, detached HEAD, ErrNotLive, probe errors, anchor refusal) can return after the task path leftover has been deleted, and whether the Add doc comment's 'before any mutation' claim matches the order."
  - "internal/fabricengine/add_taskpathleftover_integration_test.go, TestAdd_TaskPathLeftover: settle whether a row pins that a file tracked in the acting repository at the leftover's relative path keeps the directory and returns *ErrTaskPathTaken."
  - "internal/fabricengine/destroy.go, removeIgnoredTaskPath: settle whether every failure of Lstat, List, the walk and check-ignore keeps the path and lets Add return *ErrTaskPathTaken, and whether the check-ignore exit-1 branch and the delete-failure branch (error wrapping ErrStrayPathKept, path left on disk) each have a test."
  - "internal/fabricengine/addadoptonly.go ErrTaskPathTaken and ErrStrayPathKept in remove.go and doc.go: settle whether their text matches the narrowed refusal and the new source of ErrStrayPathKept in Add."
---
