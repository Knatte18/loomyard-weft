# An old driver fails on board stencils a landing installed mid-run

`stale-driver-new-stencils` · Task · bug, stencil, shed · running · Code-Burler-Review

## Seen (orch, 2026-11-17)

`launch-manifest-parity` failed at Plan-Write when `codex-wording-engine-blocks` (`40c66912`) landed mid-run:
`gatefix: fill the gatefix-body-plan prompt: stencil: include "engine-absolute-paths" has no entry in the includes`.
The landing installed new board copies of the stencils, and `gatefix-body-plan` now includes the new engine block `engine-absolute-paths`.
The driver saw that the on-disk build had changed, but spawned no successor,
since the plan writer's files in `_lym/plan/` were uncommitted.
So the old driver went on running the old binary, which has no `engine-absolute-paths` in its includes, against the new board copies.
`lym loom resume` started a fresh driver on the new build, and the run went on.

## Idea

When the build changes and a hand-over is blocked by uncommitted paths, the old binary must not render the new board copies.
Either the old driver renders from the stencils embedded in its own binary until the hand-over happens,
or it waits for the producer in flight to commit and then hands over before the next render,
or a render that names an include its binary lacks hands over to a fresh driver instead of failing the run.
