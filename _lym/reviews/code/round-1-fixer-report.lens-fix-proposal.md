```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposed fixes (F1–F4) that the fixer carried out.

no findings

I tried these cases and none broke:
- **F1:** every spliced value is an exported constant or a constant conversion, so `Long` still builds from constants.
- **F2:** reconcile discards the new second return and gets the same nil `elsewhere` as before on a failed lookup. A strand bound to no pane, or outside an own window, never enters `unread`. A successful lookup still yields `pane-elsewhere` or `pane-unlisted`. I found no consumer that treats `DownCause` as a closed set, and no stale "found nowhere" wording apart from the reconcile comment, which is accurate.
- **F3:** the `unbound` row reads `pane-unlisted` whether the lookup succeeds or fails, which matches the `DownPaneUnlisted` doc.
- **F4:** only the message literal moved into a constant. The log levels and keys are unchanged.

Tool calls:
- One grep of `internal`, `docs` and `cmd` for "found nowhere", "server-wide", "down cause" and `strand_down`, because my context lacked any search for prose that depends on the old failed-lookup behaviour.
