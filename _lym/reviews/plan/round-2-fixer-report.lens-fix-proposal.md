```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the two proposals the fixer carried out, F16 and F17.

**F16 (card 10, nilLegal only):** I found no case where it breaks.
`internal/testkit/envkit/envkit_test.go:150` pins only `Teardown.Remove` as a reported nil, and none of the other `NilSeams` users list `Teardown.Push` or `Teardown.Level`.

**F17 — [BLOCKING]**
- **Changed text:** card 1's new bullet "It touches no config, so the branch keeps its upstream setting and reads as upstream-set with a pruned ref."
  It sits with the rest of the checkOrigin and `DeleteRemoteTrackingRef` bullets.
  It is also what card 10's "pushes with no error … so `Level` reads it held" relies on.
- **Summary:** keeping the upstream while deleting the tracking ref leaves a branch that every later push refuses with a hard error.
  `checkOrigin` handles only `ErrNoUpstream`, and this state does not return it.
- **Evidence:**
  - `internal/gitrepo/blobread.go:322-333`: for a branch whose upstream is set, `UpstreamSHA` resolves the mapped tracking ref.
    When that ref is absent, it returns `gitrepo: resolve upstream …: reference not found`, not `ErrNoUpstream`.
  - `internal/fabricengine/pushclassified.go:626-630`: `checkOrigin` proceeds only on `ErrNoUpstream`.
    Any other `UpstreamSHA` error makes it return `false, upstreamErr`, before `awaitOrigin` can see that the branch is gone and its entry ended and skip it.
- **Concrete case:**
  1. A pair's code branch is deleted on origin while its entry is ended.
  2. The first pusher tick's push deletes that branch's tracking ref and keeps its upstream.
  3. The records branch gets a new commit, so card 2 reads it ahead, and the pusher pushes the pair again.
  4. The code side's `checkOrigin` now fails on `UpstreamSHA`. The error has no owed class, so it is not retried as owed.
  5. Every later pair push fails the same way.
  6. Card 10's `Push` for that pair takes the "any other failed `Push`" branch and goes Stuck. The card says this pair pushes with no error and its removal goes on.
- **What the proposal must also cover:** `checkOrigin`, or `UpstreamSHA`'s contract, must treat "upstream set, tracking ref absent" as a branch to look up on origin, not as a failure.
  Card 1 should plan that change and add a row for it. One option is a second push of the code branch after its tracking ref was deleted, which skips with nil.

Tool calls:
1. Read card 2's lines on the tracking ref and upstream, and `UpstreamSHA`'s head: I needed to know how card 2 and gitrepo read an upstream-set branch with no tracking ref.
2. Read the tail of `UpstreamSHA` and searched fabricengine for its callers: I needed the error it returns for a missing tracking ref, and `checkOrigin`'s location.
3. Read `pushJob.checkOrigin` and `awaitOrigin` in `pushclassified.go`: I needed how `checkOrigin` treats a non-`ErrNoUpstream` error.
4. Searched for `Teardown` and `nilLegal` in the envkit and battencli seam tests, and listed the `NilSeams` and `intentionallyNil` users: I needed to check whether any other list breaks under F16.
