---
round: 2
focus:
  - "Card 9 (`_lyx/plan/09-creel-deliver-marked-probe.md`), the `awaitMarkedProbe` bullet: the worker registers its next wait with `clock.After` only after a probe returns (`internal/creelengine/deliver.go`), and `creelfake.Clock.Advance` fires only timers already registered (`internal/testkit/creelfake/creelfake.go`). Settle whether the bullet names the state it waits for before `Advance` runs, so an advance cannot land between the marked pause read and the worker's `After`, and whether it states the bound on total fake time advanced against the holds, TTLs and back-offs the negative checks run under."
  - "Card 3 (`_lyx/plan/03-cherrypick-without-empty.md`): decision D2 says a pick that stops because its change is already carried is skipped and returns nil, and the constraints keep the same pick result on newer git. Settle whether every outcome the card prescribes for `Repo.CherryPick`, including a parentless commit whose tree HEAD already carries, matches `git cherry-pick --empty=drop` on the floor and on newer git, and whether each prescribed fail-loud branch has a named test step that reaches it."
  - "`_lyx/plan/00-overview.md` `## verify:`: settle whether every package a card targets, including the packages of cards 1 to 9 and their `//go:build integration` tests, is run by the untagged and the `--tags integration` lines."
  - "Card 8 (`_lyx/plan/08-reed-session-row-prime-first.md`): the tmux `S` loop orders sessions by name on tmux 3.4 and by session id on newer tmux. Settle whether each assertion the card keeps or adds for the unmarked and marked renders holds on both orders, with the fixture's session named after its worktree directory."
---

Foci cover the gating findings of round 1 and the fixes most likely to introduce new text; none restates a check the plan gate already enforces.
