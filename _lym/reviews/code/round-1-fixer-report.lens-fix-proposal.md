```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1–F11), looking for a case where a proposal is wrong or breaks something.

No findings.

- **F1, queued count on error:** a failed `measureJob` or `readHead` adds `len(jobs)-i`. I found no path where `measureJob` or `mapPackage` has already incremented `r.queued` and then returns an error, so nothing is double-counted. The plan-loop error adds only 1 for the failed package, and any jobs planned earlier go uncounted. The record still reads `Queued > 0`, so the restart rule still fires, which is all the rule needs.
- **F6, no exit record before HEAD is read:** an earlier pass's record stays on disk when a later pass refuses early. A refusing pass plans nothing, so this is the same state as before the fix. No test or caller depended on the empty-`PlannedHead` record. The `NoMappedTier` and `UnknownPackage` rows' old `writesRecord` expectation was rewritten to expect none.
- **F2, F3, F5, F7:** each row passes with the defect absent. A broken implementation would fail at least one assertion:
  - F2: the cover-builds file must not exist, and `Queued` must be 1.
  - F3: the map's `BuiltAt` is compared, and the held-lock scenario checks the skip cause and `Queued`.
  - F5: a call that skipped the locked package would return within the 200 ms window and fail the first select.
  - F7: a second slot take on a one-slot hub would block until the one-minute timeout.
- **F4:** the row can pass through the per-job HEAD read rather than the wake read if the commit lands before the pass's job-loop read. The window is tiny: `pollUntil` ticks every 10 ms, and the pass reaches the slot wait within microseconds of finishing the plan. A broken wake read would almost always fail, so I don't count this as a defect.
- **F8, F9, F10, F11:** no case found where the proposal is wrong. The child-process role keeps `SIGTERM` away from the shared test process.

Further tool calls: none, because my context already held everything this lens needed.
