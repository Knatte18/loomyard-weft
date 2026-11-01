# Reed loses the Parts-Plan-Write chair strand while its part writers run

`reed-loses-parts-chair` · Task · bug, reed, loom · running · Code-Bouncer

## Seen (orch, 2026-11-01)

In scratch worktree `psb-rv-split` of the `plan-split-bench` study, Parts-Plan-Write failed with "chair wait: reed did not track strand a6fd47d6... on 2 consecutive liveness checks; reed's strand table no longer holds this run's strand".
Earlier the chair strand `ly:psb-rv-split:parts-plan` had already vanished from `lym reed list` while its part writers still ran, and a creel reply to it could not be delivered; afterwards `lym reed status` in that worktree listed no strands at all.
Nobody ran a reed remove or down there.
The worktree was made with `lym fabric add --local` from the prime and its loom run seeded by hand, so a local pair or a hand-seeded run may be a factor.

The `plan-split-bench` study (landed 2026-11-01, `docs/research/plan-split-bench-2026-11-01/`) stopped its `run-verbs-shared` split arm at this failure, so that input has no split-versus-plan comparison; the other input, `warp-visibility`, ran the split to its end.

## Idea

Find what removes a live chair strand from reed's table during Parts-Plan-Write, and keep the chair tracked while its part writers run.
