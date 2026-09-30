---
verdict: APPROVED
rationale: "All nine round-1 findings are fixed in the current decision-record.md and support-log.md, and the fixes hold against the code: a new pair-own-weft ownership kind without the local-checkout test that remoteBranchRequest's managed-branch predicate imposes, case-split refusal remedies, tag-target fetches for cross-hub reachability, a conditional Goal, an adopt-path fast-forward probe, a named lease-race hook, and a trimmed warp rationale"
---

The report under judgment carries no `## Focus departures` section, so there is nothing to ratify or reject.

The fixer applied every finding after the reviewer wrote its report.
I judged the artifacts as they stand now, and checked each fix against the code it depends on:

- **F1 (BLOCKING), managed-branch ownership.**
  The Decisions bullet now extends `remoteBranchRequest` (`internal/fabricengine/destroy.go`) with a new ownership kind: exactly `WeftBranchName(warpBranch)`, not the primary weft branch, and no local-checkout test.
  It also adds a new dirtiness kind and a lease SHA.
  `remoteBranchRequest` today carries `ownership`, `dirtiness` and `force` but no lease, so "extend" is the correct verb.
  Constraints and Notes now agree with it.
  The unit criteria cover the ownership refusal, the checked-out-but-accepted case, and the stale lease.
  Resolved.
- **F2, refusal remedies.**
  The weft refusal distinguishes a live pair (`lyx fabric remove <slug>` on its hub) from no pair (hand-delete with `git push origin --delete <slug>-weft`, or pick another slug).
  The Unarchived-leftover criterion requires both remedies in the error.
  The support log's rejected alternative no longer claims an advantage over the chosen design.
  Resolved.
- **F3, cross-hub objects.**
  The orphan-proof decision compares SHAs first, then fetches the remote tip and the archive tag targets into `FETCH_HEAD` without creating local refs.
  A commit still missing counts as unreachable.
  The Archived-ancestor criterion exercises it.
  Resolved.
- **F4, Goal.**
  The Goal is now conditional on an archived weft leftover and a fast-forwardable warp leftover, and sends every other case to a pre-flight refusal.
  Resolved.
- **F5, rollback warp residue.**
  Scope Out names the residue, and the lease-race criterion checks only the weft rollback and origin's moved tip.
  Resolved.
- **F6, adopt path.**
  The adopt path (`weftBranchAlreadyExists` in `add.go`) gets a pre-flight `merge-base --is-ancestor` probe and a refusal, with a matching criterion.
  The unprobed assumption is gone.
  Resolved.
- **F7, lease-race seam.**
  The criterion names a test-only hook that runs between pre-flight and the deletion, and a stale-lease unit test backs it.
  Resolved.
- **F8, leanness.**
  The warp rationale is down to one clause plus the policy sentence, and the why-in-scope assumption is deleted.
  Q5 in the support log still holds the full account, and the requirement ("no remote warp deletion") stays in Decisions and Scope Out, so the completeness-before-leanness test is met.
  Resolved.
- **F9, wording.**
  The Open risks bullet now reads "a tip equal to or an ancestor of an archive tag's target".
  Resolved.

One minor point, below the finding bar.
The pre-flight weft outcomes list does not say which rule wins when the local weft branch exists and the remote tip is archived but the local branch could not fast-forward it.
The list order implies the adopt-path refusal wins, which is the conservative outcome and does not block planning.
The Decisions bullet explaining why the managed-branch kind fails partly repeats a rejected alternative.
It stays, because it carries a constraint the plan writer needs: do not reuse the managed-branch kind.
