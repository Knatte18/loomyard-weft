---
round: 2
focus:
  - "internal/hardenshed/reap_test.go, TestReap: does a row pin that a process whose Argv names the written root (not the resolved link target) is still reaped when the root sits under a link, with its Cwd outside both root forms?"
  - "internal/hardenshed/reap_test.go, TestReap: does a row reap over a root that does not exist, so filepath.EvalSymlinks fails in ReapDeps.rootProcesses in internal/hardenshed/reap.go, and assert that only the process whose Cwd is under the written root is ended and Reap returns no error?"
---

Both entries concern test coverage of the three outcomes of root resolution in `rootProcesses`.
