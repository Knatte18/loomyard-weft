---
round: 2
focus:
  - "internal/fabricengine/add_hold_test.go, TestAdd_HoldsTheCallersLock: does any row fail if Add's `AddOptions.Hold` is taken after the worktree and branch writes instead of before them, and is the release shown to run only after those writes?"
  - "internal/battencli/wire.go create closure and `refuseHeldMember`: does a test with a counting hold (not `holdNothing`) show the hold reaches `Add`, is released, and is taken around a held member's pair removal, with a hold error returned unchanged and no removal run?"
  - "internal/battenshed/create.go `HoldPrimeLock`, internal/battenshed/doc.go Worktree-Create paragraph and the comment above `CreateWorktree` in internal/battencli/wire.go: does each claim about what runs under the hold match what `Add` does under it, including its fetches and archived-leftover replace?"
  - "internal/fabricengine/add.go `AddOptions.Hold` doc and internal/battenshed/primelockwait.go header: do they state the lock order against Worktree-Teardown and the real callers of `acquirePrimeLock`?"
---

Round 1 raised no recurring keys; these entries name where to look for the test and doc claims behind the hold.
