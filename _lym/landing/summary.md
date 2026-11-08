# Re-measure the batching hook after its transcript wait: keep it

A research report in `docs/research/turns/batching-hook-remeasure-wait-2026-11-08`; no production code changes.
It reruns the scripts of `batching-hook-remeasure-2026-11-05` on the 28 darn runs with sessions after the wait (`7c77e08d`), against that report's own sample.

## Result

- The delivery gap is closed: the hook fires on 10 of 13 edit follows, where it fired on 2 of 156 before.
  It fires 34 times in 28 runs, 1.21 a run against 0.59.
- The next message batched after 8 of 19 nudged lookup and edit breach turns and 4 of 18 silent ones.
  The counts are too small to call that an effect, so the report keeps the hook and asks for a larger window, with tapestry runs, before any verdict.
- The wait adds no tokens.
  Calls above `context_tokens` show a median result latency of 1791 ms against 55 ms in the earlier sample, which is about 77 seconds over 122 messages, 2.8 seconds a run.
- Half of the calls above the line, 268 of 544, run to the two second bound and are never judged.
  The report names a shorter bound for a call of a multi-call message as the one cost to look at.

## Design decisions

- The earlier report's recommendation to remove the hook is superseded: it rested on the delivery gap, which the wait closed.
- The wait is priced from result latency in the transcripts, since the hook's own time is not logged.
  The earlier sample, which had no wait, shows no latency above the line, so the difference is attributed to the wait.
- The comparison sample is the earlier report's own, rerun with its scripts, which reproduce its figures.
  The new sample holds darn runs only, so the report reads the darn columns as the fair pair.
- The new scripts (`latency.sh`, `latency-summary.sh`, `nudge-effect.sh`, `run-all.sh`) reuse the earlier folder's extractors unchanged.
- No shorter wait bound is filed or built; the entry asked for a measurement.

## Known limits

- The window is about a day, 11 of the 39 listed runs have no session after the cutoff, and no tapestry run landed.
- Whether a session's `lym` binary carried the wait is read from the start time; the 10 edit firings show it did.
- Slice firings (14) have no breach-turn count to compare with.
