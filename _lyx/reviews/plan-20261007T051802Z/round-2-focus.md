---
round: 2
focus:
  - "Card 8 (`_lyx/plan/08-background-shell-outcome.md`): in `Run` of `internal/websterengine`, check whether the expired-shell warning and the note carry the same outcome wording on every outcome that returns a `RunResult`, including a verify-gate demotion and a Master `stuck`; and whether the card's claim about when Master's strand ends matches `Run.finalize` in `internal/shuttleengine/wait.go`, which removes the strand on a shuttle `OutcomeDone` without `KeepPane`."
  - "Card 9 (`_lyx/plan/09-gitrepo-leased-update.md`): check that each new `internal/gitrepo` method sits on the backend the package's `doc.go` assigns to its kind of read, given that `IsAncestor` runs through `runChecked` and `doc.go` records ancestry walks as measured and reverted from go-git."
  - "Card 10 (`_lyx/plan/10-reset-moves-remote-branch.md`): check that every new refusal, including a checkout rewrite that fails after the remote moved, has a planned test that reaches it, and that the card states the envelope `reset` emits when a mutation was already recorded, per `PATTERN-mutation-record`; also check that `RemoteOnlyCommits`, read by reset and by Publish, sits in a card granular for a symbol reused across packages."
  - "Cards that edit tests (cards 1 to 3, 5 to 7, 10): check each planned new test function against the existing tests of the same package, per `PATTERN-test-economy`, naming the existing test that covers the behavior."
  - "Cards 1 and 4: check that every test, fixture and doc that the planned change makes false or failing is named in a card's `Edit:` or deletion list, rather than left to an unnamed grep."
  - "`## verify:` in `_lyx/plan/00-overview.md`: for every package a card targets, check that the section runs its untagged, `integration` and `tmux` tests."
---

Round 1's open findings are re-checked against the revised cards; the focus above only steers where to look and does not limit the review.
