# creel: a new inject is charged TTL from before it was posted

`creel-inject-ttl-before-post` · Note · bug

Imported from [issue #662](https://github.com/Knatte18/loomyard/issues/662).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

A new inject is charged TTL for time that passed before it was posted.
In `internal/creelengine/deliver.go`, `worker.probe` ages every queued inject of the receiver by `now - lastProbe`.
When a worker already runs for the receiver (for example because it has unread mail) and a new inject wakes it, the new inject is charged the whole gap since the previous probe, up to the 8-second probe ceiling, or longer when the previous probe spent its time in a slow send.

Journal evidence:

- inject #34 posted 16:18:45.783, `aged` 16:18:45.813 with `consumed_s: 3` (30 ms after posting);
- inject #905 posted 16:25:08.4, `aged` 16:25:31.7 with `consumed_s: 28` (23 s after posting);
- inject #907 posted 16:26:25.6, `aged` 16:26:32.8 with `consumed_s: 9` (7 s after posting).

## Steps

1. Give a strand unread mail so its worker keeps probing on its backoff.
2. `lyx creel inject --ttl 60s <strand> "x"` while the strand is not idle.
3. Read the journal: the first `aged` record for the inject counts more seconds than have passed since its `posted` record.

## Expected

An inject's TTL counts only from its own posting (and only while the receiver is live and unpaused), so a short `--ttl` is not cut by up to several seconds.
