# Board entry `claude-md-lym-rules-out`: Move rules about lym behaviour out of this repo's CLAUDE.md into lym code and stencils

## Seen

This repo's `CLAUDE.md` holds rules about how lym itself behaves, which hold for every repo developed with lym, not only this one.
Examples: in "Docs land in the same commit", that the task that builds a design never changes its own board entry's body or scope, that the landing marks the entry done and the board prune removes it, and that no decision, card or step of a run changes the running task's own entry; in "Production", how Batten's Land row runs the hub's `deploy` command and how `alignment_pr_approve` withdraws the alignment agent's landing.

## Idea

Two kinds of rule must be kept apart (operator, 2026-10-22):

1. Rules for every use of lym, in any repo: they apply locally, to the repo the hub develops.
   They ship with lym: what lym enforces is enforced by its code, and what agents must be told lives in the stencils lym ships, so every hub gets them.
   A package's `doc.go` is documentation of what the code does, never a home for rules (operator, 2026-10-28).
2. Rules for developing LoomYard itself, here in this repo: they live in this repo's `CLAUDE.md`, such as the cgo build, `update-plugins.sh` and the push rule for `main`.

Mixing them breaks other hubs: when the quarry hub was set up as a LYMHUB repo, its orch began taking in the LoomYard repo's issues instead of quarry's own, because a rule written for LoomYard development was read as a lym rule.
`CLAUDE.md` is for one repo only, so go through it and move every rule of kind 1 out into lym; go through the stencils too for rules of kind 2 that name LoomYard where they should name the hub's own repo.

The run-lock rule in `CLAUDE.md` (no decision, card or step of a run changes the running task's own entry) is a rule of kind 1: the board's run lock enforces it, so it leaves `CLAUDE.md` with the rest.
