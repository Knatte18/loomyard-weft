---
round: 2
focus:
  - "`parentMergeTip` in internal/landingshed/parentalign.go: when it takes the local tip of a parent that is ahead only by owed landing tips, can a commit that is not an owed landing still reach the merged tip? Check the order of the lock acquisition, the local-tip read and the `ParentAheadByOwed` check, what a false answer or a failed read returns, and which callers and paths the check covers."
  - "`gateUnlocked` and `Finalize.Call` in internal/landingshed/finalize.go: for a landing that merges on GitHub, can owed, unpushed landing tips be merged into the task branch? Check how the GitHub need is read from disk before the lock-free pass, what happens when that read fails or the need only becomes known later, and whether local landings still take the owed tips."
  - "Tests in internal/landingshed (finalize_unlocked_test.go, parentalign_test.go): would each test fail if its branch broke? Cover an ahead parent that is not owed-only, a nil `ParentAheadByOwed`, an owed check that turns false once the lock is held, an owed-ahead parent under Parent-Align, a GitHub-bound landing with an owed parent, and a failed GitHub-need read."
  - "Doc comments on `parentMergeTip` and `gateUnlocked`, and any other doc or `_lym/landing/summary.md` claim about which parent tip is merged: does each match what the code does on every path, including the GitHub and failed-read exceptions?"
---

The entries cover the behaviour around which parent tip gets merged, where a change since the last review would be text no fresh reviewer has seen.
