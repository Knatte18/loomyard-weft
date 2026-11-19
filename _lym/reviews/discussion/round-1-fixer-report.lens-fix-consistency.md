```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes to the decision record and support log agree with their surroundings, and find any edit whose twin elsewhere was left unchanged.

- **MEDIUM**: D10 Bound, first sentence, against the changed release bullet and the changed Bound line about release.
  The bound still says "the server adds a revision check and drops none", but the fix added a second check the server makes: it refuses release of a task in a run status, which the CLI's `set-status` does not refuse.
  The sentence now understates what the server adds.
- **LOW**: the `PATTERN-child-processes` line in Constraints, against D5's changed opener bullet and its Bound.
  Constraints still says "the opener is waited on".
  D5 now caps that wait for a caller with a strand name and "leaves a still-running opener to finish on its own".
  The twin was not updated.
- **LOW**: the `PATTERN-daemon-wakeups` line in Constraints, against D5's new keep-warm cap.
  The fix bases the opener cap on PATTERN-daemon-wakeups' keep-warm rule (review F10), but that line names only the watch loop and the long poll.
  The capped opener wait is not listed there.
- **LOW**: support-log.md Turn 4, the line "interactions 6, bound-breaker 3 … liveness is read by the server's lock, which proves the recorded port".
  D5 no longer claims that a held lock proves the recorded port.
  It now waits until the state answers a request carrying its token, and the server removes a leftover state file after taking the lock.
  The log line contradicts the changed D5.
- **LOW**: D5's new bullet "On either path, a server that does not answer in the bound is refused with the server log's path named".
  The live path's refusal, where the lock is held and no state ever answers, has no criterion.
  The D5 criteria cover the never-answering server only on the start path ("with a server that never answers it refuses naming the server log").
- **LOW**: D9's changed state-file and launcher bullets, which add an owner-only DACL on Windows, against the `PATTERN-code-conventions` line in Constraints and D13.
  Constraints names build-tagged files only for "the platform opener".
  The Windows DACL hardening in `internal/boardserve` is a second per-platform piece, and neither Constraints nor D13 names it.

No further tool calls made.
