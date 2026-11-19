---
verdict: APPROVED
prior:
  - id: F14
    status: resolved
    answer: "D1 now names the board worktree's local branch as the constant `main` over any code default branch, and D3 hands `ensureBoardWorktree` the board name in both clone paths. D1 records the prime's records branch once in `lym.primeRecordsBranch` and has `primaryWeftBranch` read it in a one-repository hub, so the orphan sweep and `resolveManagedBranch` keep `<prime-branch>-weft`. A two-repository hub reads the board's branch as today. An acceptance criterion runs the `master` case end to end. The fixer's departure from the proposed fix, no fetch into the `--single-repo` store, is sound: `suffixWeftPrimaryBranch` adopts `origin/<prime>-weft` when it exists (clone.go:515-523), and `adoptRecordedAnchor` then refuses a rerun with another `--subpath` (clone.go:354-361)."
---

No findings.

## Checks made

- The focus question on the board's branch is settled by the record.
  `CloneHub` today passes the code store's branch to `ensureBoardWorktree` (clone.go:328), and `primaryWeftBranch` derives the prime's records branch from the board's checked-out branch (cleanup.go:337-351).
  D1 and D3 name the constant `main` for the board in both clone paths, the source `primaryWeftBranch` reads in a one-repository hub, and the unchanged two-repository answer.
- No other fabric reader derives the prime's records branch from the board's branch.
  The remaining board-branch reads (`pushclassified.go`, `originreconcile.go`, `hubpull.go`) use the board's checked-out branch as a branch to push or pull, so a board on `main` over a `master` default needs nothing more.
- The acceptance criteria hold one criterion that runs the `--single-repo` clone, a second-machine one-URL clone and the orphan sweep over a `master` default, also after `lym fabric checkout` moves the prime.
  A `main`-default fixture cannot hide a mismatch.
- The text round 2 added claims no unpinned tool behaviour.
  The no-fetch claim rests on the code paths cited above.
  The board adopting `origin/main` rests on `ensureBoardWorktree`'s remote-branch path (boardweft.go:47-54) and D1's recorded fetch evidence.
  The key's refusal way forward is a `git config` the operator runs; it adds no verb, flag or bypass that needs a bound.
