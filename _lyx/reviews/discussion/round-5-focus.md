---
round: 5
focus:
  - "Adopt's live-pair stop in the decision record's D5 step 1 and its first acceptance criterion: settle who asks the `LivePair` seam, given that `internal/fabricengine/originreconcilemoves.go` `move` returns at the `needsMove` test, true only for fast_forward and needs_replay, before it calls `pairLive`, so a branch judged ahead, diverged, rewound or rewritten never reports `BranchLive`. Check that `--drop-local` cannot move a branch under alive strands and that a test reaches the refusal through `SeamOptions.HubPaused`."
  - "D6's adopter-first repair against `finalizeEntered` in `internal/fabricengine/replay.go` and `readOriginEnded`'s `withoutFinalizeEntered`: settle whether the old machine's reconcile reads the slug ended and tears its pair down, given that a Finalize history row, which a halted Finalize leaves, makes `finalizeEntered` true, and that `parentHoldsTaskWork` stays true after the advised reset. Check that the Decision states what happens to the old machine's pair and to the step 2b records commit, and that the line-252 acceptance criterion matches."
  - "The declared `PATTERN-owner-machine` text in the decision record's PATTERN changes section, and the Constraints bullet that repeats it, against D2 and D3: settle that the wording agrees with the design for an entry with no owner recorded and for a hub with no prime owner recorded, where D3 lets a shed resume verb proceed and write no owner."
  - "D6's rule for keeping the parent lock of a run whose Finalize is running and has merged: settle which signals detect 'Finalize is running' as distinct from 'Finalize stopped' and 'run done', given `parentHoldsTaskWork` is true in all three, and that an acceptance row covers a stopped Finalize whose lock is released."
  - "D7's skip for a Finalize re-entered after its parent-side merge against D3's Spawn check in `InnerRun`: settle which re-entry paths the skip covers, and whether the landing reconcile's unreachable-origin halt is skipped for a Finalize past its parent-side merge."
  - "The halt note's `git reset --keep origin/<parent-branch>` in D6: settle what the note says about local parent commits origin lacks beyond the task's squash, and that the Decision states a bound on what the reset can discard."
---

Focus targets the six facets still open after round 4.
