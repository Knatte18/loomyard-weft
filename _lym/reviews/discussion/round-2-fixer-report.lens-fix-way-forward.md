```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, step and criterion the F14 fix added can be reached and leads to progress.

**LOW.** Changed text, D1: "A one-repository records store without the key fails closed, as an unreadable board branch does today".
Summary: the new fail-closed refusal names no way forward, though it blocks more than the sweep.
Evidence:
- `primaryWeftBranch` is read by `resolveManagedBranch` and `resolvePairWeftBranch` (`internal/fabricengine/destroy.go:828, 847`) as well as by `Cleanup` (`cleanup.go:134`).
- A one-repository hub missing `lym.primeRecordsBranch` would therefore refuse every pair's records-branch delete, not only the orphan sweep.
- Today's error text (`cleanup.go:341-343`) says only "refusing to enumerate orphan weft branches", with no remedy.
- Neither D1 nor any criterion says how the operator gets out of this state. Two options would work: write the key with `git config` in the records store naming `<prime-branch>-weft`, or re-clone the hub.
- `PATTERN-refusal-way-forward`, as Constraints states it, covers only D3's refusals, so this is a gap a reader would notice, not a contradiction.

**No finding** on the new acceptance criterion (the `master` default branch, the second-machine one-URL clone, and the sweep after `lym fabric checkout`). Each step can be reached:
- D8's builder makes a bare origin, which can take a `master` default branch.
- `lym fabric checkout` forks a missing records branch (`checkout.go:33`), so moving the prime off `master` is a step that can be carried out.
- After that move, the sweep reads the key and not the worktree.

**No finding** on the `ensureBoardWorktree` handoff in D3, for both paths:
- One-URL path: the board adopts `origin/main`, which D1's fetch refspec maps from `refs/lym/heads/main`. That takes the remote-tracking branch of `boardweft.go:47-53`.
- `--single-repo` path: the store fetches nothing, so the board takes the orphan branch of `boardweft.go:56-61`.

No further tool calls made.
