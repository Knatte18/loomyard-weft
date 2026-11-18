---
round: 1
focus:
  - "Fidelity: walk D1 to D7, D11, D12 and the Constraints in `_lym/discussion/decision-record.md` against the Card Index. Name any Decision step no card carries, in particular D2's refusals (protected-match move, `lym-bolt` present without `.lym-anchor`), D4's temporary-index records commit and the pre-step-3 merge abort, D5's marker-edit refusal and never-fast-forward rule, D11's Bolt checkout, and D12's hubforge scenarios. Also name any card scope the record does not license."
  - "Card order: cards 3 to 14 share helpers (`oneRepositoryAt`, `RecordsPathsAt`, the merge trailer, the `protected` and `operator_unpushed` classes, the `lym.primeBranch` key). Check that each card's `Uses` refs are created by an earlier card, not a later one, e.g. card 11's `Stalled` against card 14's prime-branch refusal, and card 8 against cards 2 and 7."
  - "Reachability: for each refusal or guard a card prescribes (D2 clone refusals, D4 prime-branch and absent-key refusals, D5 marker-change and unfinished-step-3 refusals, D7 replay stop and prime guard), find the named input and the named test that reaches it, and check no upstream check rejects that input first."
  - "Test economy: `TestCommitStatusSeam_Real` and other existing tests are rewritten in card 11. Check each new test function against an existing test that already covers the behaviour, and any test asserting static docs or PATTERN prose (cards 15 and 16)."
  - "Edit glyphs: for every `Edit` card, check that Intent-named symbols appear as member glyphs and that a changed signature carries the re-sign arrow. Check `Extends` refs are not rewritten, and `Custom` is not used where a label combination fits."
  - "Research steps and board: look for any card whose Intent leaves a fact to be found out during the run, and any card that edits this task's own board entry or files or promotes a task."
  - "Verify lines: check each `**Verify:**` line is a `lym gate test --wait` call with a tier list and scope form, not a raw test command."
  - "Attack surface: check the cards introducing `--single-repo`, the `lym.primeBranch` key, the `protected` completion exception in the seam, and the temporary-index records commit state in `**Intent:**` what each can skip or let through and what bounds it."
---

Seed focus only; derived from reading `00-overview.md`, the decision record and a grep of the cards.
`lens-review*` files in the plan directory were not treated as artifacts under review.
