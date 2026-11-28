# A catch-up fetch that fails on an unreachable origin still stops a run start

`catchup-fetch-stops-start` · Task · bug, dobby, fabric · running · Record-Write

## Seen

The landed `push-retry-not-halt` (`3fb0787e3`) records that a dobby catch-up whose fetch fails on an unreachable origin still stops that run's start,
and leaves to the parent whether that stays. Overnight runs then wait for an orch to run `lym batten run` again.

## Idea

Treat a catch-up fetch on an unreachable origin as a transient network stop: dobby retries the start on its own backoff and reports only past the bound,
as the pushes now do.
