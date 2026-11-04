```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as proposed (F1, F2, F4, F5, F6, F7). The F4 proposal has a problem the reviewer missed; I found nothing against the others.

**Findings**

- **MEDIUM, F4:** the reworded claim "never around an origin probe" is false against the code.
  - **Changed text:**
    - `internal/battenshed/create.go`, the `HoldPrimeLock` doc: "never around an origin probe, so one create waiting on origin's answer holds up no other".
    - `internal/battenshed/doc.go`, the Worktree-Create paragraph: "never around an origin probe".
  - **Evidence:** `internal/fabricengine/add.go:388` calls `probes.Wait()` after the pre-flight reads and before the first mutation.
    - Hold release is deferred in `Add`, so it runs at return, after that wait.
    - Add takes the hold at line 234 and releases it only at return, so the hold covers that wait.
    - On a slow or degraded origin, a create blocks at `add.go:388` while holding the prime lock. A second create waits behind it, which is the serialization the change was meant to remove.
  - **Why it matters:** the reviewer's own finding says "Only the origin probes are outside it". That is true of when the probes start, but not of when `Add` waits for them. The `AddOptions.Hold` comment (`add.go:64`, "so the hold spans no network wait of the probes") has the same flaw. The fix kept both claims and copied the wording into two more places.
  - **Proposed change:** say the probes start before the hold and overlap the pre-flight reads, so the hold covers only whatever probe time is left, bounded by the probes themselves. Do not claim it spans none. Alternatively, drop the origin-probe claim and keep only the list of fetches under the hold.

- **LOW, F4:** the fixer's `create.go` addition lists the adopt, fast-forward and replace fetches but not the probe wait at `add.go:388`. The reader is told those are the only network steps under the hold, so the list is incomplete for the same reason as above.

**Proposals with no case against them**
- F1: the hold fires before `Add` makes the worktree and its release runs from `Add`'s return-time defer, so both assertions hold. The tier-1 gate passed.
- F2: all the counts (1/1, 2/2, 1/1, no further take) match the closure's call paths.
- F5: `refuseHeldMember`'s removal runs after `Add` returns, so the pair is never created and removed at once in one closure; the comment's premise holds.
- F6, F7: no case against them.

**Further tool calls**
1. `grep` for `probes.Wait` and the probe variables in `add.go`: my context showed `Add` only up to line 252, so I had no view of where it waits for the probes.
2. `Read` of `add.go` lines 376-395: to confirm `probes.Wait()` sits before the mutations and before any hold release.
