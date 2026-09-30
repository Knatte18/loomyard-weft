# Fixer report — discussion round 1

All nine findings fixed in `_lyx/discussion/decision-record.md`, with matching entries in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits clean afterwards.

## Changes

- **F1 (BLOCKING):** the deletion now extends `remoteBranchRequest` with a new ownership kind, the pair's own weft branch: exactly `WeftBranchName(warpBranch)`, not the primary weft branch, and no local-checkout test.
  The Decision states why the managed-branch kind fails at step 12.
  Constraints and Notes now agree it is an extension, not a new shape.
  The `destroy.go` unit criteria now cover "other branch refused on ownership", "locally checked-out weft branch not refused" and "stale lease SHA fails".
- **F2 (MEDIUM):** the weft refusal remedies now distinguish a live pair (`lyx fabric remove` on its hub) from no pair (delete by hand with `git push origin --delete <slug>-weft`, or pick a different slug).
  The Unarchived-leftover criterion requires both remedies in the error.
  In `support-log.md`, the rejected alternative "plain Remove deletes by default" no longer claims an advantage the chosen design lacks; Q12 added.
- **F3 (MEDIUM):** the orphan-proof Decision compares SHAs first, then fetches the remote tip and the archive tag targets into `FETCH_HEAD`, and treats a commit still missing as unreachable.
  The third Open risks bullet names both fetches.
  Added the "Archived ancestor" criterion and Q13.
- **F4 (MEDIUM):** the Goal now makes success conditional on an archived weft leftover and a fast-forwardable warp leftover, and sends every other leftover to a pre-flight refusal with a working remedy.
- **F5 (LOW):** a Scope Out bullet names the rollback's existing warp-branch residue.
  The lease-race criterion now checks only the weft rollback and origin's moved tip.
- **F6 (LOW):** the adopt path gets a pre-flight fast-forward probe with a refusal (Decisions, Scope, new "Adopt path" criterion).
  The unprobed Auto-mode assumption is removed; Q10 is revised, and the rejected alternative "leave the adopt path unprobed" is added.
- **F7 (LOW):** the lease-race criterion names a test-only hook between pre-flight and deletion, and a stale-lease-SHA unit criterion is added; Q14 added.
- **F8 (LOW):** the warp rationale is cut to one clause plus the policy sentence.
  The why-in-scope Auto-mode assumption is deleted; Q5 already records it.
- **F9 (NIT):** the first Open risks bullet now reads "at a tip equal to or an ancestor of an archive tag's target".

## Deferred with reason

None.
