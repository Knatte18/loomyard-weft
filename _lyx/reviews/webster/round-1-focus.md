---
round: 1
focus:
  - "Comment-convention compliance against CONSTRAINTS.md's Comment Line-Break Convention (one sentence per line, break only at an independent-clause boundary, new and changed comments only): check each new or changed Go comment in internal/fabricengine/destroy.go (the leaseSHA field, the warpBranch field comment, ownedPairWeftBranch, dirtyArchivedOnRemote, archiveTag, resolvePairWeftBranch, the deleteRemoteBranch doc addition), internal/gitrepo/push.go (file header, DeleteRemoteBranchLeased), add.go, remoteleftover.go, export_test.go and the edited test-file headers, and settle whether any breaks mid-sentence at a fixed column."
  - "Edit-group blast radius of Card 3's Topology.Add pre-flight: three test files outside every card's Edit list (livestate_verbs_test.go, mutation_record_integration_test.go, origin_integration_test.go) switched to AddOptions{SkipPush: true}; settle whether each still exercises the failure or state its name and comments claim (step 11's warp push is ungated by SkipPush, the weft push is not), and whether other zero-AddOptions Add callers (hubforge.AddPair users, battencli's create-worktree row) against an absent or unreachable origin were covered by the mechanical impact check."
  - "remoteleftover.go ErrRemoteLeftover: probeWarpLeftover builds the refusal without Slug, while the field is documented as the slug Add was asked to create; settle whether the warp refusal honours the exported-field contract Card 3 sets, and whether Error()'s remedies per kind match the card's list."
  - "Step 12 replacement in add.go and deleteRemoteBranch's leased branch in destroy.go: settle that a refusal, a failed lease, or a later weft-push failure rolls back correctly, that the record carries remote_branch_deleted before the weft branch_pushed only on an observed deletion, and that the lease-race and stale-lease tests assert the moved tip survives."
  - "probeWeftLeftover's archive-proof path: settle that the fetch really writes FETCH_HEAD only (--no-tags --refmap=), that peeled ^{} targets are preferred for annotated tags, and that isAncestorOrEqual maps a locally missing commit to not-an-ancestor without swallowing other git failures."
---

Seed focus for round 1.
The review range is `git diff $(git merge-base main HEAD)..HEAD`, taken from `product.parent: main` in `_lyx/shed/fabric-readd-weft-push/status.json`; it covers Cards 1–5 plus a follow-up commit pinning `DeleteRemoteBranchLeased` in `cmd/lyx/gitrepoboundary_test.go`.
The first entry comes from the rubric's comment-convention dimension, where the repo's own rule governs.
The second covers the Edit-type mechanical check on `Topology.Add`, whose effect already appears as test edits no card named.
The rest are diff-level correctness questions on the new destructive path.
