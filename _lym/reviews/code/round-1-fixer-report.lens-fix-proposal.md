```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that were carried out as proposed (F2, F3).

**no findings** above LOW.

- **LOW**: `internal/webstercli/reset.go`, the new hand-over message (F2's proposal). It drops `fabricSyncWayForward` and puts no way forward of its own in its place, while the verb's help says "A refusal names its way forward."
  - With a mark-only hand-over, `takeoverStop` returns just `machine %q holds the run %q` (`internal/fabricengine/takeover.go:124`), so the caller gets no next step.
  - The dropped clause also held the instruction for a session lym refuses the verb from to report `status: FAILED`.
  - The success path's hand-over refusal has the same gap, and F2 asked to mirror it, so this is a weakness in the proposal rather than a new inconsistency of behaviour.

F3's two rows hold up against the code. The second `openFabric` call is the sync's. The push stub in the not-committed row would show up in `push_error` if the `!committed` guard were gone. A stubbed `ErrHandedOver` matches what `PushRun` passes through.

Outside my lens: `_lym/landing/summary.md` under `## Merge danger` says "The detached push already sent that commit there". The detached push is fire-and-forget (`internal/fabricengine/commit.go:170-171`): it may fail to start, fail to reach origin or not have run yet. The sentence, and the `two-way` door that rests on it, claim more than the code guarantees.

Further tool calls: none.
