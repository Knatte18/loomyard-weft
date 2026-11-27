---
round: 4
focus:
  - "D9, the Finalize merge-in and the removal of leftovers on a resume: where is the pre-merge task head persisted, and does a resume after a kill between the merge-in commit and the merge record write get the right base, on a first merge-in and on a later one? Check that an acceptance criterion drives both."
  - "D3 and D5, a refused landing push: for a prime parent and for a beam parent, does the named command push the code branch and clear the refused-push record? Check `lym fabric push` in the prime against internal/fabricengine/pushanchored.go. Settle too what happens when origin's parent already holds the run's work under another commit, and what happens to the landing entry of a run handed over to another machine."
  - "D5, the parent target's pusher: is the owner of every run whose landing entry a push carries read before the parent push and again before that run's done write, as well as the parent's owner? Compare handedOverStop in internal/landingshed/finalize.go (pushParent). Check the case of a beam run landed on the prime and adopted elsewhere while the push is owed."
  - "D5 and D13, the records hold: while a landing entry stands, what can each form of `lym fabric remove` (plain, --force, --remote) and its detached teardown child write or push? See internal/pairteardown/teardown.go RemovePair and claim.go settleBoardClaim, internal/fabricengine/remove.go, and the `lym fabric remove` step named in internal/runfsm/act.go."
  - "D5, pull-request landing: it checks for an owed parent push before taking the parent lock. Can a local landing record one while that run waits for the lock, and what then happens to the local parent after the GitHub merge?"
  - "D5, a parent push that origin rejects: what replaces pushParent's protected-branch way forward and its archive of a duplicate local merge, and what do later local landings onto a refused parent report?"
---
