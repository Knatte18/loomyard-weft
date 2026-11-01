# A concurrent fetch of the same ref fails a batten pass and spawns a tackler

`reconcile-fetch-race` · Task · bug, batten, fabric · running · Code-Bouncer

## Seen (orch, 2026-11-01)

At 13:03:50 a `lym batten run gate-fewer-binaries-trial --continue-pass` failed its origin reconcile: `git fetch origin +refs/heads/main-weft:refs/remotes/origin/main-weft` in the prime's weft exited 1 with `error: fetching ref refs/remotes/origin/main-weft failed: incorrect old value provided`.
That is git's answer when another process updated the same remote-tracking ref between this fetch reading it and writing it; `kde-window-placement` was landing and `board-tidy` ran at the same moment, so another fetch of `main-weft` in the same weft is the likely cause.
The pass exited 1, the driver spawned a tackler (a Claude session, about 20 s), which found nothing to do, and the loop went on.

Again the same day, at 14:46, a pass of `board-project-render` after its darn writer refused to resume: "branch board-project-render is rewound against origin; origin rewound board-project-render to 4ce42a8d, dropping 5655ca77".
Moments later `git ls-remote origin` showed `5655ca77`, the same as the local head and the tracking ref, so origin had dropped nothing: the reconcile read origin while the writer's push of `5655ca77` was still in flight.
The tackler could not tell and handed a reset-or-push question to the orch, which re-ran the pass by hand.

## Cause in the code

Fabric serialises its writes under locks (the weft write lock, the board write lock, the push lock), but `fetchMovedHeads` in `internal/fabricengine/originreconcile.go` runs a plain forced `git fetch` into the tracking refs with no lock and no retry; only the `ls-remote` before it retries, and only on a transport failure.
A remote-tracking ref is written by a fetch and by a push, so two lym processes fetching or pushing the same branch in one repository race on it.

## Idea (operator, 2026-11-01)

A fetch that writes remote-tracking refs waits its turn under a lock, as fabric's writes do, rather than retrying.
- Every lym fetch into a repository's tracking refs, the origin reconcile's included, takes the lock that already orders that repository's pushes (the push lock), since a push writes the same refs; it waits for the lock rather than failing when it is held.
- Find every fetch fabric and batten run into a shared repository's tracking refs and put each under the same lock.
- The reconcile's read of origin (`ls-remote`) and its rewind verdict take the same lock, so a push in flight finishes before origin is judged rewound.
