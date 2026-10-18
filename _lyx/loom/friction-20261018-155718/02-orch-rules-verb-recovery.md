# Recovering card 4 — a Create target the card itself removes

Card 4 names `.scratch/orch-rules-move.md` as its one `Create:` target and then tells the implementer to remove it by its literal path once `lyx board upsert` has read it.
Webster's create-not-done check then looks the target up through quarry after the batch, finds nothing and rejects the batch, though the board note was filed correctly.
The recovery left the git-ignored draft in place, refilled from `lyx board get --body`, so the target resolves; the card's removal step and the check cannot both hold.
A planner writing a card whose only effect is outside the repo, such as a board write, has no target type that the check accepts for a transient draft.
