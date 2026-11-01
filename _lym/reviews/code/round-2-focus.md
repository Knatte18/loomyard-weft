---
round: 2
focus:
  - "internal/gitrepo: for each of Fetch, FetchWithArgs, Pull, PushRebaseFree, DeleteRemoteBranch, DeleteRemoteBranchLeased and UpdateRemoteBranchLeased, settle whether a test fails deterministically when the method stops waiting on a held remote-refs lock."
  - "internal/fabricengine/originreconcile.go, judgeAgainstOrigin: settle whether the remote-refs locks stay held from the origin read through every branch verdict, and whether a test fails if they are released after the fetch."
  - "internal/fabricengine/originreconcile.go, ReconcileOrigin: settle whether every return, including the unreachable-origin, error and judge-only paths, frees both repositories' locks, and whether a test covers each."
  - "PATTERN.md remote-refs-lock entry against internal/fabricengine/archive.go (tagAndPushArchiveTip, resolveArchiveTip): settle whether the entry's wording matches what the code does for the tag push, and whether the FETCH_HEAD read happens under the same lock hold as its fetch."
---

Focus targets the lock's enforcement and its hold span, the two places the first round found the diff least pinned.
