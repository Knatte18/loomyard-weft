# Hardener: an autonomous behavior-review campaign that replaces crucible

`hardener` · Notes · feature

## Goal

Replace `crucible`, the hand-run review-and-fix loop, with `Hardener`.
Hardener is an on-demand campaign after loom that runs a live-substrate module in a sandbox, reviews its behavior round after round, and fixes what it finds until the gates are green.
Then retire `crucible/`, which still writes to `_mill/` and runs its rounds through hand-spawned `crucible-reviewer-<effort>` agents.

## Starting point

- `manifest/designs/hardener.md` (DRAFT): `Tenter` is the round loop on `Treadle`, and `Hardener` is `Shed` wrapping `Tenter` with its own Preflight.
  Rounds respawn every time instead of keeping a persistent thread, and a `progress-judge` runs before and after each round.
- `crucible/README.md` is the method Hardener automates.

## Open before Discussion

- What the handoff must carry losslessly besides the finding-recurrence ledger.
- Where the line falls between the Go scaffolding and the LLM's orchestrator role.
- Whether the round agent reuses the `burler` package or only its prompt template.
- How much of the sandbox provisioning lyx automates.

## Done when

- `lyx` can start a Hardener campaign against a module and run it unattended to convergence or a round cap.
- `crucible/` is deleted, and the design's durable content lives in package docs.

## Notes

- One of the Someday items on `manifest-retire`'s triage list (`hardener`). Landing this entry removes it from that list.
