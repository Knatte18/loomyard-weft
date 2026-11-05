# Raddle on a Sketch fails to start because the motif run that made it already completed under the same slug

`raddle-same-slug-completed` · Note · bug, batten, dobby, loom

## Seen (orch, 2026-11-05, twice)

Promoting a Sketch gives the same entry recipe raddle under the same slug.
For `reed-slack-relay` and again for `creel-cross-hub-mail`, dobby could not start the raddle run: battencli reported the slug `has already completed` and named `lym batten cleanup --apply`, since the motif run that made the Sketch left its run directory under `_lym/shed/<slug>`.
The orch ran the cleanup each time and released the task again.

## Idea

The promotion of a Sketch to raddle, or batten at Seed of a raddle run, clears the finished motif run's directory of that slug, as the cleanup does, so the raddle run starts with no hand step.
