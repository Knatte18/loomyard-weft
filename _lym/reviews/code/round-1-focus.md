---
round: 1
focus:
  - "internal/hardenshed/reap.go rootProcesses: does matching both the written root and its EvalSymlinks form close the issue #164 gap, and can the resolved-root match end a process the written-root match would not, such as an unrelated process under a shared link target?"
  - "internal/hardenshed/reap.go rootProcesses: is the EvalSymlinks failure path handled soundly, where the root is missing or the error is swallowed, and does it fall back to the written root only?"
  - "internal/hardenshed/reap_test.go EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget: does the orphan name the root only through its link-resolved cwd, so the test would fail without the fix, and does it assert independent values rather than recomputing the code's own resolution?"
  - "internal/hardencli/cli_integration_test.go TestSandboxVerbs: does the orphan started with Dir = root still exercise the clear path on Linux, and does the test still check what its name and comments state?"
  - "Root-Cause trailer on commit 19d52874: does it name the defect the diff removes, and is the decision record's issue text (resolve symlinks or compare both, plus a test with a link cwd and no root on the command line) fully carried, with no unlicensed change?"
  - "internal/hardenshed/doc.go and the rootProcesses doc comment: do they match the behavior as the diff leaves it, and follow the repository's comment conventions?"
---

Scope is small: one production function, one new test, one restored integration-test shape, and two doc edits.
The answer key is the decision record's `## Goal` entry alone; its other sections are empty.
