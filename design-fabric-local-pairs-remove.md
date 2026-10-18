# Remove local-only fabric pairs, and run every remote operation async

`fabric-local-pairs-remove` · Note · enhancement, fabric, batten

## Seen

`batten-start-speed` builds local-only fabric pairs (`fabric add --local`, a local-only branch mark in `gitrepo` with a push refusal, pushes that skip such branches, and local modes of `fabric remove` and `fabric cleanup`): about 630 lines of code, 1360 with tests, plus a local-mark check in the status-push spawn entry.
The mode came from an orch recommendation in `weaver` that batch pairs never reach origin, which was wrong: a batch pair's commits are part of the implementation, and an agent continuing the work on another machine must see what a batch left half done.
The time it saves is small: in the sandbox hub `lyx fabric add` took about 1.1 s, nearly all of it two synchronous `git ls-remote` origin probes, with local work about 50 ms and its push already async; `lyx fabric remove --remote` took 6 to 7 s of network, at teardown, where nothing waits on it.

## Decisions (operator, 2026-10-18)

- Remove the local-only pair mode once `batten-start-speed` has landed: `--local`, the local-only branch mark and its push refusal, the skipped-local reporting in pushes, the local modes of remove and cleanup, and the local-mark check in the status push.
- Every operation against a remote runs async, so no lyx command waits on the network.
  This includes `fabric remove`'s 6 to 7 s of network at teardown (force-with-lease push, archive-tag push, weft branch delete).
- One exception may stay synchronous: the check whether a pair's branches already exist on origin, since a branch found there is fetched and the work continued on, and that is worth waiting for.
  Today that is `fabric add`'s `git ls-remote` probes; they may run in parallel to cut the wait.

## Open

- Which remote operations are synchronous today besides `fabric remove`'s (force-with-lease push, archive-tag push, weft branch delete); an audit of the call sites belongs to the task that builds this.
- How an async remote failure reaches the operator or the orch.
