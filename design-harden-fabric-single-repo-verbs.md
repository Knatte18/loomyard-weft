# Second hardener campaign on fabric, one-repository layout, with the fabric verbs

`harden-fabric-single-repo-verbs` · Task · bug, fabric · running · Describe

## Focus

Second hardener campaign on the one-repository hub layout that 57015a52 added: one origin holds the code under `refs/heads/` and `refs/tags/`, and the records and board under `refs/lym/heads/` and `refs/lym/tags/`.
Everything else in fabric, the two-repository layout included, is out of scope, and no round is steered outside this focus.

The first campaign, `harden-fabricengine`, could not drive the regions behind the mutating `lym fabric` verbs, since a task session was refused them.
`harden-sandbox-fabric-verbs` (4037bdf36) now lets a hardener round run every fabric verb inside its own `.lym/harden-sandbox` hubs.
This campaign drives what the first left unverified, on one-repository sandbox hubs (`lym harden sandbox new --single-repo`), with real git and real `lym` verbs:

- **The sandbox exception itself**: every mutating fabric verb runs inside the round's own sandbox hub, and stays refused against the real hub and another worktree's sandbox root.
- **Joining**: a second `lym fabric clone <url>` of a seeded origin takes the one-repository layout, gets the refspecs and `branch.main.merge refs/lym/heads/main`, and sees the board.
- **Namespace leaks on every path**: add, push, pull, sync, replay, hand-over, checkout, remove, teardown, reconcile, `cleanup --apply` and `prune --apply` never put a records ref under plain `refs/heads/` or `refs/tags/` on origin.
- **Archive tags**: the archive tag of a discarded task branch lands under `refs/lym/tags/`, never plain `refs/tags/`; it was once seen there in a sandbox trial.
- **Two clones at once**: pushes and pulls from two clones of the same origin, a board write from each, a lease that loses.
- **Remote delete**: lym's own delete of a missing namespaced branch records no deletion; of a present one removes it under `refs/lym/heads/`.
- **A pair verb killed mid-operation**: what origin and the hub hold after each kill, and whether the verbs a reader runs name a way forward.
- **A killed seed and its way forward**: after the seeding `lym` is killed, `lym board sync` and `lym fabric clone --single-repo ... --reset` lead to a working hub; `seed-killed-way-forward-loops` fixes the wording first.

Every finding states the repro steps that showed it.
