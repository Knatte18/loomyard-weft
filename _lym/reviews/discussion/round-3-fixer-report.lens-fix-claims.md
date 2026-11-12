```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added or rewrote against the code at HEAD. Two findings, one MEDIUM and one LOW.

- **MEDIUM**, D5: "So `weftDirtyStop`, which guards a weft only behind origin, no longer stops on them (`internal/fabricengine/pull.go`)."
  This misdescribes `weftDirtyStop`, and the fix's own Bound contradicts it.
  `weftDirtyStop` builds the dirty stop for two paths. One is the behind-only weft (`pull.go:415`). The other is the replay's `ErrReplayDirty` (`pull.go:440-445`).
  The Bound says the live-pass stop is a `*WeftStopError` naming `lym fabric commit`. That is `weftDirtyStop`'s output on the replay path, so `weftDirtyStop` can still stop on a map.
  What holds is narrower: the behind-only branch of `settleUnmovedWeft` (`len(ahead) == 0`) is no longer taken, because the commit leaves the weft ahead.

- **LOW**, D5: "The commit puts the weft one commit ahead."
  The maps commit adds one commit to whatever local commits the weft already holds over origin. The weft is at least one commit ahead, not exactly one.
  The conclusion (replay rather than fast-forward) still holds: `settleUnmovedWeft` replays whenever `ahead` is non-empty (`pull.go:406-432`).

These claims hold against the code:
- The verdict lives in `internal/loomcli/prepush.go`, unexported, as `skipsCheck`, `guard` and `stopReason`.
- `pairteardown` builds the reconcile's seams (`landingdeps.go:756-760`), and `hubgeom.PauseGeometry` places the pause record. Both import `fabricengine`, which confirms the reason given for putting the reconcile and pause read in each caller.
- The skips are pushes or git off, the prime, a pause, and Finalize (`prepush.go:94-110`).
- The pass-throughs are a reconcile failure, unreachable origin, and a failed read of whether the parent holds the work (`prepush.go:135-143`, `223-227`).
- `lym tender resume` appears only in the `EntryStopped` reason (`prepush.go:237`).
- `replayRequest` uses `dirtyTrackedOn(touched)`, and `touched` holds every path of every commit ahead (`replay.go:441-461`).
- The dirtiness refusal becomes `ErrReplayDirty` (`destroy.go:2172-2173`, `2187-2188`), and `pull.go:440-456` turns it into a `*WeftStopError`. Its way forward is `lym fabric commit`, then re-run `lym fabric pull`. The branch stays unmoved (`replay.go:419-420`).

Tool calls:
- One grep for `PullWeft` in `recordsgit.go` and `cherryPickOnto` in `destroy.go`. My context lacked how a failed pull reaches the replay and how a dirtiness refusal becomes `ErrReplayDirty`.
