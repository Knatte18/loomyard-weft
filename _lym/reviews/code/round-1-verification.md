---
round: 1
entries:
  - key: own-share-misses-call-processes-outside-guarded-tree
    status: closed
    finding: F1
    reason: "457101cab: memoryGuard.kills sums RSS over g.own() (newMemoryGuard sets proc.TreeSample(os.Getpid()), used by tier and report guards), otherHolders excludes that set, and TestMemoryGuard has the 6000 MB sibling row wanting an ordinary kill."
---
