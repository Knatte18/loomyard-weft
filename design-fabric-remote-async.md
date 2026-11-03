# Every remote operation runs async, and local-only fabric pairs stay an explicit choice only

`fabric-remote-async` · Task · enhancement, fabric, batten

Depends on: [`remote-moved-before-push`](design-remote-moved-before-push.md)

## Seen

`batten-start-speed` builds local-only fabric pairs (`fabric add --local`, a local-only branch mark in `gitrepo` with a push refusal, pushes that skip such branches, and local modes of `fabric remove` and `fabric cleanup`): about 630 lines of code, 1360 with tests, plus a local-mark check in the status-push spawn entry.
The mode came from an orch recommendation in `weaver` that batch pairs never reach origin, which was wrong: a batch pair's commits are part of the implementation, and an agent continuing the work on another machine must see what a batch left half done.
The time it saves is small: in the sandbox hub `lym fabric add` took about 1.1 s, nearly all of it two synchronous `git ls-remote` origin probes, with local work about 50 ms and its push already async; `lym fabric remove --remote` took 6 to 7 s of network, at teardown, where nothing waits on it.

## Decisions (operator, 2026-10-18)

- Every operation against a remote runs async, so no lym command waits on the network.
  This includes `fabric remove`'s 6 to 7 s of network at teardown (force-with-lease push, archive-tag push, weft branch delete).
- One exception may stay synchronous: the check whether a pair's branches already exist on origin, since a branch found there is fetched and the work continued on, and that is worth waiting for.
  Today that is `fabric add`'s `git ls-remote` probes; they may run in parallel to cut the wait.

## Since the landing of `batten-remote-order` (2026-11-03)

`fabric remove` now pushes the weft tip alone before the archive tag, retries an archive-tag push refused with "missing necessary objects", and deletes the weft branch on origin under a lease on the tip it archived.
`Add` and the task-branch discard take a per-slug teardown lock exclusive, and every pair push takes it shared, so a hand `lym fabric add` that meets a running removal is refused.
So a removal waits on one more network push than before, and an async version must keep this order: weft tip, then archive tag, then the leased delete, all under the teardown lock (`PATTERN-slug-remote-order`).

## Decisions (operator, 2026-11-03)

- The async remote work follows the detached-child pattern the pair push already uses (`DetachedPush` in `internal/fabricengine/teardownlock.go`).
- The task starts with an audit of every remote call site, and makes each one async except the origin-branch probe above.
  A call site that does not fit the detached-child pattern is a stop for the parent, not a redesign inside this run.
- An async remote failure is written to the fabric log and sent to the run's orch as one creel notice line naming the slug, the operation and the way forward.
- An async removal hands the teardown lock to its detached child, which holds it exclusive until the last remote step ends, so the order above holds while `fabric remove` returns at once.
- The local-only pair mode goes now: no flow uses it.
  Remove `fabric add --local`, the local-only branch mark and its push refusal, the skipped-local reporting in pushes, the local modes of `fabric remove` and `fabric cleanup`, and the local-mark check in the status push, with their tests and docs.
