---
round: 1
ledger:
  - key: rollback-tests-vacuous-after-preflight
    rounds: [1]
    status: resolved
  - key: warp-leftover-refusal-missing-slug
    rounds: [1]
    status: resolved
  - key: lease-race-test-missing-moved-tip-assert
    rounds: [1]
    status: resolved
  - key: destroy-go-fixed-column-comments
    rounds: [1]
    status: resolved
  - key: remoteleftover-header-fixed-column
    rounds: [1]
    status: resolved
  - key: delete-remote-branch-leased-doc-fixed-column
    rounds: [1]
    status: resolved
  - key: new-test-comments-fixed-column
    rounds: [1]
    status: resolved
  - key: add-go-preflight-comment-breaks
    rounds: [1]
    status: resolved
  - key: skippush-switch-unexplained
    rounds: [1]
    status: resolved
  - key: destroyremote-header-either-case
    rounds: [1]
    status: resolved
---

In round 1 the reviewer raised F1–F10 and still approved.
The headline finding was F1: three broken-origin rollback tests passed vacuously once Add's new pre-flight `ls-remote` refused first.
F2 and F3 were contract and assertion gaps in the new warp refusal and the lease-race test.
F4–F10 were comment line-break and wording fixes under CONSTRAINTS.md's Comment Line-Break Convention.
One fix commit per finding landed before this judge pass, and each one checks out in the diff.
All ten entries are resolved.
A pre-existing fabriccli integration failure in Cleanup's archive path is outside this diff and is not tracked here.
