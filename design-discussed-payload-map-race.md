# boardengine: TestDiscussedPayload crashes with concurrent map writes

`discussed-payload-map-race` · Task · bug, board

Imported from [issue #289](https://github.com/Knatte18/loomyard/issues/289).

## What happened

A card gate in `internal/boardengine` crashed with `fatal error: concurrent map writes` in `TestDiscussedPayload`.
The batch being implemented did not touch that package, and a rerun of the same gate passed.

## Why it is abnormal

A concurrent map write is a data race in the test or in the code under it.
It fails gates at random for tasks unrelated to the package.

## What to do

Run `TestDiscussedPayload` repeatedly with the race detector, find the unsynchronized map, and fix it.

Provenance: task `push-retry-not-halt`, note `05-pusher-interval.md`.
