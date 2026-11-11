---
round: 2
focus:
  - "In internal/fabricengine, check every way-forward line the diff builds with lymVerbDir (stopWayForward, anchorStopWayForward, dirtyWayForward, weftDirtyStop, upstreamAnchorStop, settleUnmovedWeft, the push steps): does an existing test assert each one against an anchored worktree, so reverting the conversion fails a test?"
  - "In internal/fabricengine/pull.go and anchorstop.go, check that every `lym fabric` step in a stop or way-forward text names the anchored directory while its git steps stay at the worktree root."
  - "In internal/fabricengine/failedpush.go, check that lymVerbDir's empty-root and invalid-or-unreadable-marker fallbacks are each pinned by a test row."
  - "In internal/fabricengine/doc.go, check that the added sentence about way-forward directories claims no more than the lines the diff converts."
---

Round 2 focuses on whether the fixes landed for the anchored-directory conversions and their tests.
