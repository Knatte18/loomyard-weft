```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out (fix-proposal lens).

**BLOCKING: F5's leases don't close the race once F6 allows replacing a leftover.** Changed text: D3, the leased seed-push list and the Bound sentence "It replaces a namespaced records branch only through a push leased on the tip it observed, and its board push lands only while origin still holds no board branch."
Here is the case:
1. Clone A checks its preconditions and finds no records branch and no board.
2. A pushes `refs/lym/heads/<prime>-weft` at tip TA. Its lease on absence holds.
3. Clone B checks its preconditions. It sees TA and no board, which D3 now treats as a leftover to replace.
4. B pushes its records branch, leased on TA. The lease holds, and B's anchor commit replaces A's.
5. A pushes its board, leased on absence. It lands.
6. B pushes its board, leased on absence. The lease fails, and B tears down its hub directory only.

Origin is left with A's board beside B's records branch, and B's anchor may record a different `--subpath`. That is the exact state F5's premise names, so the premise is still unresolved.

The F6 "replace the leftover" choice is what reopens it. A records branch with no board looks the same whether an earlier attempt left it or a concurrent clone pushed it a moment ago. The rationale "a hub that reads as one on origin always carries its anchor" is also false here, because it carries B's anchor.

Two ways to resolve it:
- On a failed board lease, B's teardown restores the records branch it replaced, leased on its own tip.
- Or D3 says this residue is possible and states what the second clone of the resulting origin does with it.

**MEDIUM: F2's dropped `.lym-anchor` arm lets a half-cleaned f06acf39 origin through `--single-repo`.** Changed text: D3's fourth refusal bullet, which now refuses only "an origin holding the branch `lym-bolt`".
The case is an operator who deleted `lym-bolt` but left `.lym-anchor` and `_lym` tracked on the code branches. That origin passes every arm:
- it holds no namespaced board branch;
- its default branch holds no `.lym-warp`, since f06acf39 kept that on `lym-bolt`;
- it has no `lym-bolt`.

The prime then checks out a tracked `_lym` directory where the `_lym` junction into the records store has to go. It also checks out a tracked root `.lym-anchor` in the code worktree. The removed arm was the guard against this.

The other option the review offered, refusing a root `.lym-anchor` with no `.lym-warp`, would catch a pre-binding two-repository weft (`internal/fabricengine/clone.go:401` comment). So the fixer was right to reject that one.

A sound scoped arm exists: with `--single-repo` only, refuse a default branch that tracks a root `.lym-anchor` or `_lym`. A two-repository weft never reaches it, because the clone refuses a weft that holds `.lym-warp` first. A pre-binding weft lacks `.lym-warp` but is a records repository, which nobody passes to `--single-repo`. The way forward stays the same cleanup.

**LOW: F3's `ls-remote` check is stated for the namespace only, though D4 runs one code path for both layouts.** Changed text: D4, "The plain delete therefore first runs `git ls-remote <url> refs/lym/heads/<branch>-weft`".
D2 makes the namespace the identity, `refs/heads/`, in a two-repository hub, and D4 says the two-repository code paths run in both layouts. Deleting by full name in a two-repository hub, `refs/heads/<branch>-weft`, also exits 0 on a missing ref. The comment at `internal/gitrepo/push.go:313` says so: "a full refs/heads/ name deletes a missing ref silently".
So either the `ls-remote` check runs in both layouts, or the two-repository short-name delete stays. D4 states neither, and as worded the two-repository teardown loses its absent-ref answer.

Tool calls:
- grep of `internal/fabricengine/clone.go` for the one-URL warp-binding handling, to see how a one-URL clone treats an origin with no `.lym-warp`. My context lacked that part of `CloneHub`.
