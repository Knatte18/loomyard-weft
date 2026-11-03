---
round: 2
exclude_lenses: []
focus:
  - "Resume of a run whose task worktree is gone, D1: check that the adopt-only `Add` call in `internal/fabricengine/add.go` cannot be refused by its non-empty `Anchor` refusal on a live weft (lines 397-401) or by `ErrSourceDirty` on tracked dirt in the prime (lines 239-246), and settle what the call passes."
  - "Land way forward, D2: for each way `Add` answers or refuses when the pair is gone (not-live answer, `ErrRemoteLeftover` in `internal/fabricengine/remoteleftover.go`, records-worktree-present refusal at `add.go:328-331`), settle whether the named steps restore the child's run records, and how `lym batten run` treats the failed status at Land on the re-run."
  - "D4 and D6 push and lease: check D4's weft branch push against `PATTERN-push-both-sides` as the task declares it, and settle which origin tip D6's lease names after D4's own push, and whether `Cleanup` in `internal/fabricengine/cleanup.go` archives an origin tip it deletes."
  - "D3 cancel: check what `lym batten cancel` does to a record standing at Land and to the slug-scope owner pause, given `childReachedParent` in `internal/battencli/cancel.go` and the Finalize stop in `internal/landingshed/finalize_owner.go`."
---

Every lens stays: each ran in round 1 and carried a finding at MEDIUM or worse.
