---
round: 3
focus:
  - "In decision-record.md D9 and D1, does the moved-parent check read the same parent ref that Finalize's catch-up merge-in and parent-side merge use (the local parent branch, merged with no fetch), so that a commit made only on the local parent branch after Merge-In is detected at Publish, PR-Gate and Finalize and cannot land through the catch-up without merge review? Check the Finalize retry condition and the acceptance criterion on a commit arriving between Finalize's check and its parent-side merge against the same question."
  - "In decision-record.md D1, where Merge-In fast-forwards the local parent branch: does the Decision name which worktree it moves (the parent pair's checked-out one), whether it runs before or after the parent lock is held, and a Stuck with a way forward for a dirty parent worktree, a merge in progress and a missing parent pair, with the bound that it never touches local commits?"
  - "In decision-record.md D1, for a hub where the parent push is skipped (the told push skip that leaves the local parent ahead of its upstream after a landing): does a second Merge-In on the same parent still have a way forward that does not require the skipped push?"
---

Each entry restates the sites it depends on.
