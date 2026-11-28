# A missing local origin is not classed as unreachable by push or origin reconcile

`missing-origin-unreachable` · Task · bug, fabric, git · done

Imported from [issue #294](https://github.com/Knatte18/loomyard/issues/294).

## What happened

An origin that is missing on disk is not treated as unreachable:

- A removed bare repository used as origin is classed `other` by the push failure classification, not as transport.
- A remote pointing at a missing directory makes the origin reconcile return an error instead of reporting origin unreachable.

Tests of an owed push or an unreachable origin had to use a refused connection URL instead.

## Why it is abnormal

An origin that cannot be reached for a path reason, such as a missing or unmounted directory, takes a different route from one that refuses a connection.
It may therefore stop a run where an owed push would be expected.

## What to do

Decide how a missing local remote should be classed.
Either treat it as unreachable in both places, or document that only network failures count.

Provenance: task `push-retry-not-halt`, notes `06-pusher-target-kinds.md`, `12-finalize-local-landing.md`.
