# Measure the effect of the batching breach hook

`batching-hook-effect` · Note · research, claude, shuttle, tools

## Idea (operator, 2026-11-02)

Once the batching breach hook (`batching-breach-nudge`) has landed, measure over time whether it cuts the drift it targets, and tune its config thresholds from what is seen.

## Measurement

The baseline is the 28-session sample behind `batching-breach-nudge`: single-lookup turns that follow another single lookup above 150k context, 128 turns and 34.7M context tokens, and edits spread over consecutive turns, 39 turns and 15M, about a third of the sample's 147M.
Measure the same two figures, and the context tokens of every session, over a later sample of sessions with the hook on, with the transcript tooling under `tools/` (`tokencount`) extended where it lacks the breach counts.
Compare the two samples by recipe and role, since a darn writer and a plan writer drift differently.
Also read how often the hook fired and how often the next message after a firing batched, from the hook's own log, and how many firings were on a lookup that had to be sequential.

## Bounds

No new agent run is made: the work reads the transcripts and hook logs of sessions that already ran.
The later sample is taken at least two weeks after the hook lands, with the hook's thresholds unchanged over that window.
It runs as darn with profile `docs` in its own worktree, so the report lands as a commit, and its report names any threshold change it recommends as a note of its own.

## Timing

Not before `batching-breach-nudge` has landed and the later sample exists; it waits on that landing through `depends_on` once promoted.
