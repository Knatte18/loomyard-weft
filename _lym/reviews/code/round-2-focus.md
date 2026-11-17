---
round: 2
focus:
  - "internal/fabricengine/pairbranch.go (originTaskBranch, originPairBranch, pairBranchNames) and internal/fabricengine/add.go (Add under AdoptOnly and AdoptLocal): when branch_prefix is a per-machine key, does a pair pushed under prefix a/ resolve to its own branch a/<slug> on a machine whose prefix is b/, in the lookup and in the re-adopt path of internal/battencli/readopt.go, and does a new pair still use the current prefix?"
  - "internal/fabricengine/pairbranch.go slugOfBranch and its callers in pairparts.go (TaskBranchSlugs), cleanup.go (cleanupOrphan) and cleanupremotewarp.go (deleteLeftoverUnderLock, keptReason): for a branch made under another machine's prefix, is the derived slug the pair's own slug, so the teardown lock path matches the one the pair's teardown takes?"
  - "Tests added for the other-prefix lookup and slug derivation: do they cover the new behavior without duplicating an existing test, and do the doc comments and the change description's open-risks text match what the code now does?"
---

Focus targets the fix commits of the round's design findings.
