---
round: 4
focus:
  - "Settle whether every Info-or-above log line reached on each iteration of a loop, watch tick, wake hook or poll has a D11 verdict row and, where guarded, a card change. Follow the whole per-iteration call graph, not the first callee: include readers that a hook or seam callback calls in another package, such as the `internal/reedengine` state-load path that `reapplyLayout` and `repairNames` call every tick, the `internal/shuttleengine` run-record, wait-marker, activity and session-state readers that loomcli's and battencli's loop and seam callbacks call, and `parentlock` `Store.Acquire`'s per-poll release path."
  - "For each `keep` verdict row in the D11 table in `_lyx/plan/00-overview.md`, settle whether its stated rationale holds on every branch of the code path, including the failure branch that repeats while the triggering condition persists and the fallback path taken when a sibling query fails, such as `repairSessionName` in `internal/reedengine/namerepair.go` and `runResizePinsLocked` in `internal/reedengine/apply.go`."
  - "For each guard a card prescribes with a recovery branch (`Recover` or a recovery Info line), settle whether a named test or test row fails the seam enough times to log, then succeeds, so the recovery branch is reached; a row whose seam fails on every call does not reach it. Check cards 9 through 15, including `reviewWaitNote` in `internal/battenshed`."
---

The focus targets the D11 sweep method, the keep-verdict rationales and test reach of recovery branches.
