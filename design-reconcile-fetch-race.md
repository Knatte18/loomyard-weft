# A concurrent fetch of the same ref fails a batten pass and spawns a tackler

`reconcile-fetch-race` · Note · bug, batten, fabric

## Seen (orch, 2026-11-01)

At 13:03:50 a `lym batten run gate-fewer-binaries-trial --continue-pass` failed its origin reconcile: `git fetch origin +refs/heads/main-weft:refs/remotes/origin/main-weft` in the prime's weft exited 1 with `error: fetching ref refs/remotes/origin/main-weft failed: incorrect old value provided`.
That is git's answer when another process updated the same remote-tracking ref between this fetch reading it and writing it; `kde-window-placement` was landing and `board-tidy` ran at the same moment, so another fetch of `main-weft` in the same weft is the likely cause.
The pass exited 1, the driver spawned a tackler (a Claude session, about 20 s), which found nothing to do, and the loop went on.

## Idea

The fetch in batten's origin reconcile, and any other fetch of a shared ref in a prime's warp or weft, retries once or twice when git reports `incorrect old value provided`, or serialises those fetches under a lock, so a race between two lym processes never fails a pass or costs a tackler.
