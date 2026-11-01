```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: checking that the F2 fix covers every return of `judgeAgainstOrigin`. F1 is a decision finding, which the parent handles, so I did not check it.

No findings.

F2's set is the returns of `judgeAgainstOrigin` while the locks are held (`internal/fabricengine/originreconcile.go`, lines 761–810). Each return and what pins it:

| Line | Return | Pinned by |
|---|---|---|
| 763 | Error from the second `HoldRemoteRefs` | Not pinned. It is the lock-acquire error itself, and the first lock's `defer` frees it. |
| 770 | Origin unreachable | The "frees both remote-refs locks on every return" subtest. |
| 773 | `ls-remote` refused | The same subtest, via its no-repository row. |
| 788 | `fetchMovedHeads` error | Pinned by the fixer's new assertion in the failed-fetch subtest. |
| 800 | `judgeBranch` error | Not pinned. The review accepted this, since it needs a forced git failure and the shared `defer` makes it the same path. |
| 810 | Success | The verdict-hook subtest and the judge-only row. |

The 763 return was not in the review's list, and the review did not ask for it.

Tool calls:
1. `grep` on `judgeAgainstOrigin` in `originreconcile.go`: my context did not show the function body, which I needed to list its returns.
