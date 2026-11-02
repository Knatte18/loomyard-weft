# Measure the effect of the batching breach hook

`batching-hook-effect` · Note · research, claude, shuttle, tools

## Idea (operator, 2026-11-02)

Measure over time whether the batching breach hook (task `batching-breach-nudge`, landed 2026-11-02) cuts the drift it targets, and tune its config thresholds from what is seen.

## The hook as landed

A PostToolUse hook on Read, Grep, Glob, Edit and Write runs `lym shuttle hook-batching claude <state file>` and adds one `additionalContext` line on a breach; it never denies.
It judges three breaches: a single-lookup chain, a Read slice, and an edit-only chain.
Its thresholds are the `batching_nudge` block of `claude.yaml`: `context_tokens` 150000, `lookup_run` 3, `edit_run` 2, `slice_lines` 2000, `max_nudges` 5, and an `enabled` switch beside one per breach.
A run copies the values at its start into a per-run `batching-nudge.json` beside its events file, with the nudge count, so a running session keeps the values it started with.

## Measurement

The baseline is the 28-session sample behind `batching-breach-nudge`: single-lookup turns that follow another single lookup above 150k context, 128 turns and 34.7M context tokens, and edits spread over consecutive turns, 39 turns and 15M, about a third of the sample's 147M.
Measure the same two figures, and the context tokens of every session, over a later sample of sessions with the hook on, with the transcript tooling under `tools/` (`tokencount`) extended where it lacks the breach counts.
Compare the two samples by recipe and role, since a darn writer and a plan writer drift differently.
Also read how often the hook fired and how often the next message after a firing batched, from the per-run `batching-nudge.json` nudge counts and the transcripts, and how many firings were on a lookup that had to be sequential.

## Bounds

No new agent run is made: the work reads the transcripts and hook state of sessions that already ran.
The later sample is taken at least two weeks after the hook landed, so not before 2026-11-16, with the hook's thresholds unchanged over that window.
It runs as darn with profile `docs` in its own worktree, so the report lands as a commit, and its report names any threshold change it recommends as a note of its own.

## Timing

The hook has landed; the work waits only for the later sample to exist.
