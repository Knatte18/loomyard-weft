---
round: 1
focus:
  - "internal/landingshed/parentalign.go parentMergeTip: ParentAheadByOwed is read with no parent lock, and the local tip is read later under it. Settle what tip is merged when the parent changes between the two reads (the owed push completes, a landing is made but not yet recorded, a local commit that is no landing appears, the lock wait is long), and whether each outcome is sound."
  - "Agreement between the two reads of one parent: compare what ParentAheadByOwed (internal/loomcli/landingdeps.go, fabricengine.AheadByOnly, the push ledger) judges with what ReadParentTips reads for Deps.ParentBranch in the lock-free pass and with the head the locked pass compares gatedTip against. Settle whether both always concern the same branch, repository and upstream."
  - "A task branch that now holds the parent's unpushed landing commits: trace what happens afterwards on every path out of gateUnlocked and the locked pass (gate failure and fixer, halt and resume, a lost race, a landing that merges on GitHub or has an open pull request, a task branch push, an owed push later refused or the parent reconciled by rewriting). Check the merge record and overlap values, and whether unpushed parent commits can reach origin or GitHub through the task branch."
  - "Prose that describes which parent tip a lock-free merge takes or what an ahead parent means: Deps.ParentAheadByOwed and Deps.ReadParentTips docs in deps.go, the landingshed package doc, the comments on Finalize.Call, gateUnlocked and parentMergeTip, docs/overview.md and pattern/. Settle whether any is now incomplete or false."
  - "internal/landingshed/finalize_unlocked_test.go: settle which new branches have a test that fails if they are wrong, namely an ahead parent that is not owed-only still merging the upstream tip, Parent-Align still merging the upstream tip of an owed-ahead parent, and a nil ParentAheadByOwed. Check whether an existing test already holds each before calling it uncovered."
  - "_lym/landing/summary.md against the code: check each behaviour claim, including that other ahead parents are refused by the locked pass's sync as before, and the before-the-fix test claim. Check the two-way door call and the blast radius of landing."
---

Seeded from the committed diff and the amended board entry.
The change is small and sits in one function, so the focus is on what the newly merged local tip affects after the merge.
